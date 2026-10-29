

```
Shader "Custom/URP/Avatar_GridSlice"
{
	Properties
	{
		[PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {}
		[MainColor] _Color ("Tint", Color) = (1,1,1,1)
		_HalfLen ("half len",Range(0,0.5)) = 0.12
		_Width ("width",Range(0,0.25)) = 0.05
		_Delay("delay",Range(0,0.1)) = 0.05
		_Degress("degress",Range(0,180)) = 45
		_Cutoff ("Alpha Cutoff", Range(0,1)) = 0.1
	}

	SubShader
	{
		Tags
		{	
			"Queue"="Transparent"
			"RenderType"="Transparent"
			"RenderPipeline"="UniversalPipeline"
			"IgnoreProjector"="True"
			"CanUseSpriteAtlas"="True"
		}

		Cull Off
		ZWrite Off
		Blend One OneMinusSrcAlpha
		
		Pass
		{
			Name "SpriteUnlit"	
			Tags { "LightMode"="UniversalForward" }

			HLSLPROGRAM
			#pragma vertex Vert
			#pragma fragment Frag

			#include "Assets/BuiltInAsset/Shader/SDF/SDFWrap.hlsl"
			#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
			float DrawSpindleLine(float2 p, float2 dir, float halfLen, float width)
			{
				float2 n = float2(-dir.y, dir.x);
				float x = dot(p, dir);
				float y = dot(p, n);
				float t = saturate(abs(x) / halfLen);
				float localWidth = width * sin((1.0 - t) * 3.1415926 * 0.5);
				float insideLine = step(abs(y), localWidth);
				float insideLen = step(abs(x), halfLen);
				return insideLine * insideLen;
			}
			
			struct Attributes
			{
				float3 positionOS : POSITION;
				float4 color : COLOR;
				float2 uv : TEXCOORD0;
				UNITY_VERTEX_INPUT_INSTANCE_ID
			};

			struct Varyings
			{
				float4 positionHCS : SV_POSITION;
				float4 color : COLOR;
				float2 uv : TEXCOORD0;
				UNITY_VERTEX_INPUT_INSTANCE_ID
				UNITY_VERTEX_OUTPUT_STEREO
			};

			CBUFFER_START(UnityPerMaterial)
				float4 _Color;
				float _HalfLen;
				float _Width;
				float _Delay;
				float _Degress;
				float _Cutoff;
			CBUFFER_END

			TEXTURE2D(_MainTex);
			SAMPLER(sampler_MainTex);

  

			Varyings Vert(Attributes IN)
			{
				Varyings OUT;
				UNITY_SETUP_INSTANCE_ID(IN);
				UNITY_TRANSFER_INSTANCE_ID(IN, OUT);
				UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(OUT);
				OUT.positionHCS = TransformObjectToHClip(IN.positionOS);
				OUT.uv = IN.uv;
				OUT.color = IN.color * _Color;
				return OUT;
			}

			half4 Frag(Varyings IN) : SV_Target
			{
				UNITY_SETUP_INSTANCE_ID(IN);
				float2 uv = IN.uv;
				float theta = radians(_Degress);
				
				float2 p = uv - float2(0.5f,0.5f);
				float s = sin(theta);
				float c = cos(theta);
				
				float x = p.x;
				float y = p.y;
				p = float2(x * c - y * s, x * s + y * c);
				float2 VerticalDir = float2(0,1);
				float2 HorizonDir = float2(1,0);
				float spacing = 1.0f / 9.0f;
				float SDF = 0;
				
				[unroll]
				for(int i = -4;i < 5;i++)
				{
					float offset = spacing * i;
					float dist01 = abs((float)i) / 4.0;
					float curWidth = max(0.0, _Width - _Delay * abs((float)i));
					float curHalfLen = _HalfLen * saturate(1.0 - dist01 * 0.35);
					float val = DrawSpindleLine(p - float2(0.0, offset),
													HorizonDir,
													curHalfLen,
													curWidth);
					SDF = max(val,SDF);
				}
				
				[unroll]
				for(int i = -4;i < 5;i++)
				{
					float offset = spacing * i;
					float dist01 = abs((float)i) / 4.0;
					float curWidth = max(0.0, _Width - _Delay * abs((float)i));
					float curHalfLen = _HalfLen * saturate(1.0 - dist01 * 0.35);
					float val = DrawSpindleLine(p - float2(offset,0.0),
															VerticalDir,
															curHalfLen,
															curWidth);
					SDF = max(val,SDF);
				}
				
				half4 texColor = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, IN.uv);
				clip(texColor.a - _Cutoff);
				return half4(0,0,0,1.0-SDF);
			}
			ENDHLSL
		}
}

FallBack Off
}
```


![[GridSlice.png|408]]