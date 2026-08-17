---
date: 2026-07-07
course:
tags:
  - Unity
  - URP
  - Shader制作
---
## 要件
UI上で実行中にmaskを生成し、表示範囲を動的に調整するような要望です。

## 原理
UnityではMaskコンポーネントが提供してくれています。Mask は子要素の形状を親要素のものに制限します。Maskの動作原理はまずStencilに比較値を書き込んでおき、子要素が描画される時にStencil内の比較値と比較して条件が満ちる部分だけを描画する。
そのため、カスタマイズシェーダーで動的に形を計算してStencilに比較値を書き込めば、実行中にMask表示範囲を調整できるようになります。



```hlsl
Shader "Custom/UI/URP_CustomMaskShape"
{

Properties
{
	[PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {}
	_UpSlope ("UpSlope", Float) = 0.3
	_DownSlope ("DownSlope", Float) = 0.1
	_UpHeight ("UpHeight", Float) = 0.3
	_DownHeight ("DownHeight", Float) = 0.1
	
	[HideInInspector] _StencilComp ("Stencil Comparison", Float) = 8
	[HideInInspector] _Stencil ("Stencil ID", Float) = 0
	[HideInInspector] _StencilOp ("Stencil Operation", Float) = 0
	[HideInInspector] _StencilWriteMask ("Stencil Write Mask", Float) = 255
	[HideInInspector] _StencilReadMask ("Stencil Read Mask", Float) = 255
	[HideInInspector] _ColorMask ("Color Mask", Float) = 15
}

  

SubShader
{
	Tags
	{
		"Queue" = "Transparent"	
		"RenderType" = "Transparent"
		"RenderPipeline" = "UniversalPipeline"
		"IgnoreProjector" = "True"
		"CanUseSpriteAtlas" = "True"
		"PreviewType" = "Plane"
	}

	Stencil
	{
		Ref [_Stencil]	
		Comp [_StencilComp]
		Pass [_StencilOp]
		ReadMask [_StencilReadMask]
		WriteMask [_StencilWriteMask]
	}

	Cull Off
	ZWrite Off
	ZTest [unity_GUIZTestMode]
	ColorMask [_ColorMask]

	Pass
	{
		Name "UI_CustomMask"
		HLSLPROGRAM
		
		#pragma target 2.0	
		#pragma vertex Vert
		#pragma fragment Frag
		#pragma multi_compile_local _ UNITY_UI_CLIP_RECT
		#pragma multi_compile_local _ UNITY_UI_ALPHACLIP

		#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
	
	struct Attributes
	{
		float4 positionOS : POSITION;
		float2 uv : TEXCOORD0;
	};
	
	struct Varyings
	{
		float4 positionHCS : SV_POSITION;
		float2 uv : TEXCOORD0;
	};
	
	CBUFFER_START(UnityPerMaterial)
		float _UpSlope;
		float _DownSlope;
		float _UpHeight;
		float _DownHeight;
	CBUFFER_END
	
	Varyings Vert(Attributes IN)
	{
		Varyings OUT;
		OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
		OUT.uv = IN.uv;
		return OUT;
	}

  

	half4 Frag(Varyings IN) : SV_Target
	{
		float2 pos = IN.uv - float2(0.5,0.5);
		float upLine = tan(_UpSlope) * pos.x + _UpHeight;
		
		float downLine = tan(_DownSlope) * pos.x + _DownHeight;
	
		clip(min(pos.y - downLine, upLine - pos.y));
	
		#ifdef UNITY_UI_ALPHACLIP
		clip(1 - 0.001);
		#endif
	
		return half4(1,1,1,1);
	}

	ENDHLSL
}
}
FallBack "UI/Default"
}
```




