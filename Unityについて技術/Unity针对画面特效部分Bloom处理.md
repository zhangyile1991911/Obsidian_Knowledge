---
date: 2026-05-25
project:
tags:
  - Unity
  - URP
  - Bloom
  - 日语
---
## 要件
全画面にブルームを適用するより、画面内にエフェクトが存在する部分のみにブルームを適用する

## 原理
- エフェクトを指定テクスチャーに描画させる
- 指定テクスチャーにブルームを適用する
- 指定テクスチャーを現在カメラの描画結果と混ぜあわせる
  
## 対応方法

Q: エフェクトを指定テクスチャーに描画させる
A: Unity内の<mark style="background: #ABF7F7A6;">RenderObject</mark>という既存機能があり、このコードを踏襲して出力先を他のテクスチャーに入れ替える

Q: 指定テクスチャーにブルームを適用する
A: Unity内に既存ブルーム結果を一致するため、エンジン内Shaderを使用する

Q: 指定テクスチャーを現在カメラの描画結果とブレンドする
A: 無し

## エフェクトをテクスチャーに描画する
- RenderToTexturePassクラスを作成する
- RenderToTextureFeatureクラスを作成する
### 1. ScriptableRenderPassの子クラスを作成する

```csharp

public class RenderToTexturePass : ScriptableRenderPass
{

    //指定テクスチャー
    public RTHandle ExternalTargetColor;

    public override void RecordRenderGraph(RenderGraph renderGraph,ContextContainer frameData)
    {
        using(var builder = renderGraph.AddRasterRenderPass<PassData>(k_PassName,out PassData passData,profilingSampler))
        {
            TextureHandle colorTarget = renderGraph.ImportTexture(ExternalTargetColor);

            //今回出力先を入れ替える
	       builder.SetRenderAttachment(colorTarget,0,AccessFlags.ReadWrite);
        }
    }

}
```

### 2. ScriptableRendererFeatureの子クラスを作成する

```csharp
public class RenderToTextureFeature : ScriptableRendererFeature
{
    private RTHandle m_TargetTexture;
    //エフェクトをテクスチャーに描画するパス
    private RenderToTexturePass m_RenderPass;

    public override void Create()
    {
        m_RenderPass = new RenderToTexturePass();
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        //新規テクスチャーを制作する
        var cameraDesc = renderingData.cameraData.cameraTargetDescriptor;
        //アルファチャンネルを確保する
        cameraDesc.colorFormat = RenderTextureFormat.ARGBHalf;
        cameraDesc.depthStencilFormat = GraphicsFormat.None;
        cameraDesc.msaaSamples = 1;
        cameraDesc.bindMS = false;
        RenderingUtils.ReAllocateHandleIfNeeded(
            ref m_TargetTexture,
            cameraDesc,
            FilterMode.Bilinear,
            TextureWrapMode.Clamp,
            name:"GlobalEffectTarget");

        m_RenderPass.ExternalTargetColor = m_TargetTexture;
        renderer.EnqueuePass(m_RenderPass);
    }

}

```

### 3. 結果確認
![[only_effect_bloom.png|485]]


## Bloom処理を適用する

### 新規ScriptableRenderPass
- ScriptableRenderPassを継承する
- Unity６に既存Bloomシェーダーを利用する
```csharp
public class PartialBloomPass : ScriptableRenderPass
{
	public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
	{
		//必要な情報を取得する
		UniversalCameraData cameraData = frameData.Get<UniversalCameraData>();
        UniversalRenderingData renderingData = frameData.Get<UniversalRenderingData>();
        UniversalLightData lightData = frameData.Get<UniversalLightData>();
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
        //現在バファのサイズに基づいて半分、四分のテクスチャーを生成する
        var srcDesc = resourceData.activeColorTexture.GetDescriptor(renderGraph);
        int textureWidth = Mathf.Max(srcDesc.width >> downres);
        int textureHeight = Mathf.Max(srcDesc.height >> downres);
        //テクスチャー生成数を確定する
        int maxSize = Mathf.Max(textureWidth, textureHeight);
        int iterations = Mathf.FloorToInt(Mathf.Log(maxSize, 2f) - 1);
        int mipCount = Mathf.Clamp(iterations, 1, m_Setting.MaxIterations);
        
        var down = new TextureHandle[mipCount];
        var up = new TextureHandle[mipCount];
        //一時的テクスチャーを生成する
        for (int i = 0; i < mipCount; i++)
        {
            mipDesc.name = "_BloomMipDown" + i;
            down[i] = renderGraph.CreateTexture(mipDesc);

            mipDesc.name = "_BloomMipUp" + i;
            up[i] = renderGraph.CreateTexture(mipDesc);

            mipDesc.width = Mathf.Max(1, mipDesc.width >> 1);
            mipDesc.height = Mathf.Max(1, mipDesc.height >> 1);
        }
        //バックアップ用テクスチャー
        var activeColorCopy = renderGraph.CreateTexture(copyDesc);
        using (var builder = renderGraph.AddUnsafePass<PassData>("Bloom Core", out var passData))
        {
	        //今回利用テクスチャーを入れる
	        passData.source = renderGraph.ImportTexture(m_Setting.source);
            builder.UseTexture(passData.source, AccessFlags.Read);

            passData.targetColor = resourceData.activeColorTexture;
            builder.UseTexture(passData.targetColor,AccessFlags.ReadWrite);

            passData.cameraColorCopy = activeColorCopy;
            builder.UseTexture(passData.cameraColorCopy,AccessFlags.ReadWrite);
			
			for (int i = 0; i < mipCount; i++)
            {
                builder.UseTexture(down[i], AccessFlags.ReadWrite);
                builder.UseTexture(up[i], AccessFlags.ReadWrite);
            }
			
			//描画処理の始まり
			builder.SetRenderFunc(static (PassData data, UnsafeGraphContext context) =>
            {
                CommandBuffer cmd = CommandBufferHelpers.GetNativeCommandBuffer(context.cmd);
                //現在カメラの結果をコピーしておく
                Blitter.BlitCameraTexture(cmd, data.targetColor, data.cameraColorCopy);
                
				//マテリアリティのパラメーターを配置する
                var bloomMat = data.bloomMat;
                bloomMat.SetVector(_ParamsId, data.bloomShaderParams);
                CoreUtils.SetKeyword(bloomMat, "_BLOOM_HQ", false); 
                CoreUtils.SetKeyword(bloomMat, "_ENABLE_ALPHA_OUTPUT", false);
                for (int i = 0; i < data.mipCount; i++)
                {
                    var upMat = data.upsampleMats[i];
                    upMat.SetVector(_ParamsId, data.bloomShaderParams);
                    CoreUtils.SetKeyword(upMat, "_BLOOM_HQ", false);
                    CoreUtils.SetKeyword(upMat, "_ENABLE_ALPHA_OUTPUT", false);
                }
				
                const RenderBufferLoadAction load = RenderBufferLoadAction.DontCare;
                const RenderBufferStoreAction store = RenderBufferStoreAction.Store;
                //Prefilter
                Blitter.BlitCameraTexture(cmd, data.source, data.down[0], load, store, bloomMat, 0);
                //横ブラーと縦ブラー
                TextureHandle lastDown = data.down[0];
                for (int i = 1; i < data.mipCount; i++)
                {
                    Blitter.BlitCameraTexture(cmd, lastDown, data.up[i], load, store, bloomMat, 1);
                    Blitter.BlitCameraTexture(cmd, data.up[i], data.down[i], load, store, bloomMat, 2);
                    lastDown = data.down[i];
                }
                //Upsample
                for (int i = data.mipCount - 2; i >= 0; i--)
                {
                    var lowMip = (i == data.mipCount - 2) ? data.down[i + 1] : data.up[i + 1];
                    var highMip = data.down[i];
                    var dst = data.up[i];

                    var upMat = data.upsampleMats[i];
                    upMat.SetTexture(_SourceTexLowMipId, lowMip);
                    Blitter.BlitCameraTexture(cmd, highMip, dst, load, store, upMat, 3);
                }

                var bloomOut = (data.mipCount == 1) ? data.down[0] : data.up[0];
                data.compoiteMat.SetTexture(_EffectTex,data.source);
                data.compoiteMat.SetTexture(_BloomFinalTexId,bloomOut);
                data.compoiteMat.SetVector(_BloomCompositeParamsId,data.bloomCompositeParams);
                //結果を混ぜ合わせる
                Blitter.BlitCameraTexture(cmd,
                    data.cameraColorCopy,
                    data.targetColor,
                    RenderBufferLoadAction.Load,
                    RenderBufferStoreAction.Store,
                    data.compoiteMat,
                    0);
            });
        }
	}
}
```


### 最後のブレンド
```hlsl
float4 CompoiteBloomFinal(Varyings input) : SV_Target
{
	float2 uv = UnityStereoTransformScreenSpaceTex(input.texcoord);
	half4 srcColor = SAMPLE_TEXTURE2D_X(_BlitTexture, sampler_LinearClamp, uv);
	half4 srcEffect = SAMPLE_TEXTURE2D_X(_EffectTex, sampler_BloomTex, uv);
	half3 bloom = SAMPLE_TEXTURE2D_X(_BloomFinalTex, sampler_BloomFinalTex, uv).rgb;
	
	 float3 sceneWithFx = srcColor.rgb * (1.0 - srcEffect.a) + srcEffect.rgb;
	 half intensity = _BloomCompositeParams.x;
     half3 tint = _BloomCompositeParams.yzw;
     
     float3 outCol = sceneWithFx + bloom * tint * intensity; 
    return float4(outCol,src4.a);
}
```

## 最終結果確認
![[final_bloom.png|598]]