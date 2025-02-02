---
layout: post
title: 강주성 TA 포트폴리오
---
## 목차

1) [렌더링](#렌더링)

[메인캐릭터 그림자](#배경에-더-선명한-캐릭터-그림자-드리우기)

[노멀편집](#얼굴-normals-제어하기)

[외곽선 제어](#얼굴-안쪽의-외곽선-마스킹)

[캐스트 쉐도우 제어](#Cast-Shadow-제어하기)

[눈썹 먼저 그리기](#머리카락보다-앞에-있는-눈썹)

2) [리깅](#리깅)

[블렌더 세컨더리본](#세컨더리-본-anim-bake를-위한-python-스크립트)

[유니티 세컨더리본](#게임엔진에서-세컨더리-본-제어)

3) [사용자정의 URP](#사용자정의-URP)

[2D 머리카락 그림자](#2D-머리카락-그림자)

[안드로이드 테스트](#안드로이드-테스트)

4) [모델링](#모델링)

모치다 아리사 작업물



# 렌더링



## 배경에 더 선명한 캐릭터 그림자 드리우기

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/arknight%20example.png?raw=true">

[이미지 출처 영상](https://www.youtube.com/embed/4660-3DJydA?si=B8S5bSzeGk5Bes65&amp;start=17)

명일방주-엔드필드 시연 영상에서 영감을 얻어, 전체 그림자맵 용량을 아끼면서 캐릭터 그림자를 선명하게 띄우는 방법을 구상

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/how%20to%20render%20main%20character%20shadow.png?raw=true">

그림자맵은 카메라로 렌더한다는 것에 착안, 메인 캐릭터 위에 직선광과 똑같은 각도의 Orthographic 카메라를 배치

카메라에 부착된 MainCharacterShadowCaster.cs는 셰이더에게 카메라 뷰프로젝션 매트릭스와 뎁스맵을 전송함

    using UnityEngine;
    using UnityEngine.Rendering;
    
    static int mainCharacterShadowmapId = Shader.PropertyToID("_MainCharacterShadowmap");
    static int mainCharacterShadowMatrixId = Shader.PropertyToID("_MainCharacterMatrix");
    
    private RenderTexture depthTex;
    private Camera cam;

    void Awake()
    {
        cam = GetComponent<Camera>();
        depthTex = cam.targetTexture;
    }
    private void LateUpdate()
    {
        Shader.SetGlobalTexture(mainCharacterShadowmapId, depthTex);
        Shader.SetGlobalMatrix(mainCharacterShadowMatrixId, MatrixConversion.GetVP(cam));
    }

    public static Matrix4x4 GetVP(Camera cam)
    {
        bool d3d = SystemInfo.graphicsDeviceVersion.IndexOf("Direct3D") > -1;
        
        Matrix4x4 V = cam.worldToCameraMatrix;
        Matrix4x4 P = cam.projectionMatrix;
        if (d3d) {
            // Invert Y for rendering to a render texture
            for (int i = 0; i < 4; i++) {
                P[1,i] = -P[1,i];
            }
            // Scale and bias from OpenGL -> D3D depth range
            for (int i = 0; i < 4; i++) {
                P[2,i] = P[2,i]*0.5f + P[3,i]*0.5f;
            }
        }
        return P * V;
    }

 
셰이더에서 뎁스맵과 프로젝션 상 픽셀 위치를 비교하여, 뎁스맵의 픽셀이 월드 픽셀보다 앞에 있으면 그림자에 가려진 것으로 처리함

    
<details>
	<summary>셰이더 메인 캐릭터 그림자 처리 부분(클릭 시 펼쳐집니다)</summary>
	<div markdown="1">
	
    	float4 ProjectUVFromWorldPos(float4x4 projection, float3 worldPos) {
				float4 projVertex = mul(projection, float4(worldPos.x,worldPos.y, worldPos.z, 1) );
				return ComputeScreenPos(projVertex);
			}

			float2 ProjectionUVToTex2DUV(float4 projUV) {
				return projUV.xy / projUV.w;
			}

			bool ClipUVBoarder(float2 uv) {
				if (uv.x < 0 || uv.x>1) return true;
				if (uv.y < 0 || uv.y>1) return true;

				return false;
			}

			bool ClipBackProjection(float4 projUV) {
				if (projUV.z < 0) return true;
				
				return false;
			}

			float DepthFromDepthmap(Texture2D depthMap, sampler sampler_depthMap, float2 projectedUV, float bias) {
				return SAMPLE_TEXTURE2D(depthMap, sampler_depthMap, projectedUV).r*bias;
			}

			float DepthFromDepthmap(Texture2D depthMap, sampler sampler_depthMap, float2 projectedUV) {
				return SAMPLE_TEXTURE2D(depthMap, sampler_depthMap, projectedUV).r;
			}

			bool ClipProjectionShadow(float depthFromPos, float depthFromMap, float calculationOffset) {
				if (depthFromPos - depthFromMap < calculationOffset) return true;

				return false;
			}
    
    TEXTURE2D(_MainCharacterShadowmap);
    SAMPLER(sampler_MainCharacterShadowmap);
    float4x4 _MainCharacterMatrix;
    
    
    float3 MainCharacterLightShadow(float3 positionWS) {
    
        float4 projUV = ProjectUVFromWorldPos(_MainCharacterMatrix, positionWS);
        float2 uv = ProjectionUVToTex2DUV(projUV);
    
        if (ClipUVBoarder(uv)) {
            return -1;
        }
        if (ClipBackProjection(projUV)) {
            return -1;
        }
    
        float dfp = DepthFromProjection(projUV);
        float dfd = DepthFromDepthmap(_MainCharacterShadowmap, sampler_MainCharacterShadowmap, projUV, 1);
    
        return float3(uv.x, uv.y ,1 - ClipProjectionShadow(dfp, dfd, -0.0001));
    }
    
    Light GetMainLight(float3 positionWS, half4 shadowMask, float shadowCastOffset)
    {
        Light light = GetMainLight();
    
        float3 virtualWorldPos = positionWS + (light.direction * shadowCastOffset);
    
        float4 shadowCoord = TransformWorldToShadowCoord(virtualWorldPos);
    
        light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, shadowMask, _MainLightOcclusionProbes);
    
    #if defined(MAIN_CHARACTER_SHADOW_ON)
        float3 detailShadow = MainCharacterLightShadow(virtualWorldPos);
        if (detailShadow.z > -0.1) {
    
            light.shadowAttenuation = min(light.shadowAttenuation, detailShadow.z);
          
        }
    #endif
    
        #if defined(_LIGHT_COOKIES)
            real3 cookieColor = SampleMainLightCookie(positionWS);
            light.color *= cookieColor;
        #endif
    
        return light;
    }


</div>
</details>

결과물 

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/main%20character%20shadow.gif?raw=true">

-일반 그림자맵 크기 512, 메인캐릭터용 뎁스맵 크기 256

-맵크기는 줄이면서 오히려 그림자는 선명해지는 효과를 얻음

 
## 얼굴 Normals 제어하기

카툰렌더링에서 그림자를 만화적으로 지게 만들기 위해 노멀을 편집.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/normal%20work%203.png?raw=true">

-3D모델의 그림자는 버텍스에 주어지는 법선 벡터로 그려집니다.

-그래서 만화적으로 간략화된 그림자가 나오도록 버텍스의 법선백터를 조정합니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/face%20normal%20before%20after.png?raw=true">

노멀이 편집된 상태에서 표정 애니메이션을 주니 노멀이 다시 계산되어 망가지는 문제 발생.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/normal3.png?raw=true">

해당 문제를 해결하기 위해서 커스텀 스크립트 구현

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/normal4.png?raw=true">

위 작업에 사용된 Custom Skinned Mesh Renderer 스크립트

[Custom Skinned Mesh Renderer 스크립트](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Visuals/CustomSkinning/CustomSkinnedMeshRenderer.cs) 



	...
    for (int i = 0; i < blendshapeWeights.Length; i++)
        {
            int frameCount = mesh.GetBlendShapeFrameCount(i) - 1;
            mesh.GetBlendShapeFrameVertices(i, frameCount, deltaVertices, /*deltaNormals*/null, null);
            for(int j =0; j < vertices.Length; ++j)
            {
                if (syncShapeValueWithSkinned && origSkinned != null)
                {
                    blendshapeWeights[i] = origSkinned.GetBlendShapeWeight(i)*0.01f;
                }
                    
                float weight = (blendshapeWeights[i] + (i == blendshapeWeights.Length - 1 ? -1 : 0));
                vertices[j] += deltaVertices[j] * weight; 
                //normals[j] += deltaNormals[j] * weight;    // 원본 매시의 노멀을 유지하기 위해 블랜드셰이프의 노멀 계산을 비활성화
            }
        }
	...

엔진내장 Skinned Mesh Renderer와 Custom Skinned Mesh Renderer 비교

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/facial%20normal%20compare.gif?raw=true">

더 나아가 Job System+Burst Compile로 스크립트를 멀티스레드 상에서 기동시켜 최적화를 진행하였습니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Jqh19h7k2AM?si=Q8B78PZmN3B9D70M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



　
　
## 얼굴 안쪽의 외곽선 마스킹

표정 변화를 위해 버텍스 애니메이션을 주니, 메쉬가 접히면서 쓸모없는 외곽선이 생겼습니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/outline%20compare.png?raw=true">

셰이더와 스탠실로 지워보겠습니다.

Vertex Color를 이용

R채널 : 항상 그려지는 외곽선 렌더(코, 얼굴 이외 외곽선)

B채널 : G채널 마스킹 전용 면(ColorMask 0, 최종 카메라에 렌더되지 않음)

G채널 : B채널에 의해 마스킹 된 외곽선 렌더(얼굴외곽)

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/vertex painting.png?raw=true">



마스킹 적용 전(상)과 의도대로 외곽선이 마스킹 된 모습(하)

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/outline before after.png?raw=true">
　

 
## Cast Shadow 제어하기


-라이팅에 의해 생기는 그림자는 현재
  
  a) 버텍스 노멀에 의해서 생성되는 lit 그림자

  b) 앞의 물체가 햇빛을 가려서 다른물체에 드리우는 cast shadow

두 종류 입니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/shadow%20explain.png?raw=true">

이미지 출처: https://myranaito.medium.com/how-to-shade-d7d42b55caa5

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/wanna%20do%20with%20shadow.png?raw=true">

얼굴에 cast shadow를 적용하게 되면 코나 입술에 의해서 만화적인 느낌을 해치는 그림자가 드리우게 됩니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/cast%20shadow%20topic.png?raw=true">

-RealtimeLights.hlsl를 수정. shadowCoord를 GetMainLight()안에서 계산하여, 얼굴 면의 그림자공간 위치 값을 머리카락과 콧날보다 앞에 오게 함

    Light GetMainLight(float3 positionWS, half4 shadowMask, float shadowCastOffset)
    {
        Light light = GetMainLight();
    
        float3 virtualWorldPos = positionWS + (light.direction * shadowCastOffset);
    
        float4 shadowCoord = TransformWorldToShadowCoord(virtualWorldPos);
    
        light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, shadowMask, _MainLightOcclusionProbes);
    
        #if defined(_LIGHT_COOKIES)
            real3 cookieColor = SampleMainLightCookie(positionWS);
            light.color *= cookieColor;
        #endif
    
        return light;
    }


얼굴 그림자를 의도대로 제어하면서 Cast Shadow 양립

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/controlling%20cast%20shadow.gif?raw=true">







## 머리카락보다 앞에 있는 눈썹

눈썹이 머리카락보다 위에 오는 만화적 표현을 위해, 눈썹의 버텍스 월드위치를 카메라 방향으로 앞으로 당기기.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/eyebrow.png?raw=true">

적용된 셰이더 코드


	VertexPositionInputs GetVertexPositionInputs(float3 positionOS, float _depthForward)
	{
	    VertexPositionInputs input;
	    input.positionWS = TransformObjectToWorld(positionOS);

	    float3 viewDirection = GetWorldSpaceNormalizeViewDir(input.positionWS);//_WorldSpaceCameraPos.xyz - input.positionWS;

	    input.positionWS += viewDirection * _depthForward;

	    input.positionVS = TransformWorldToView(input.positionWS);
	    input.positionCS = TransformWorldToHClip(input.positionWS);

	    float4 ndc = input.positionCS * 0.5f;
	    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
	    input.positionNDC.zw = input.positionCS.zw;

	    return input;
	}
	Varyings LitPassVertex(Attributes input)
	{
		...
		VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz, _DepthForward);
		...
	}


　


# 리깅

 
## 블렌더에서 파이썬 스크립트를 이용해서, 자동화된 어깨 / 겨드랑이 / 엉덩이 본 만들기

팔다리 움직임을 감지해서 자동으로 움직이는 보조 본을 파이썬 스크립트로 구현해봤습니다.

3DS MAX, 마야 에서도 동일한 구현이 가능합니다.


   
## 어깨본

-사람이 팔을 위로 들면 해부학적으로 어깨는 뒤로 젖혀지게 됩니다. 그래서 팔을 위로 들면 어깨본이 뒤로 젖혀지도록 스크립트로 구현했습니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/shoulder_secondary.gif?raw=true">

-윗팔본이 어느 각도냐에 따라서, 어깨본이 어떤각도가 되야 인체해부학에 맞고 보기 좋은지 수동으로 저장함.

-애니메이터가 윗팔본을 움직일 때마다, 윗팔본의 각도를 감지함, 

-위에서 감지된 각도에 따라 미리 저장해놓은 보기좋은 어깨본의 각도로 어깨본이 자동으로 움직임.

## 겨드랑이본

-어깨본과 같은 원리 입니다. 미리 윗팔본의 각도에 따라 보기좋은 겨드랑의 본의 각도를 저장합니다. 

이후 윗팔본의 움직임에 따라 저장된 값으로 자동으로 움직입니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/armpit.gif?raw=true">


## 엉덩이 본

-어깨본, 겨드랑이본과 같은 원리이지만, 윗팔본 대신 허벅지본을 기준으로 움직입니다.

<img src ="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/hip.gif?raw=true">


## 계산식

1. 팔의 각도에 따라서 보기좋은 세컨더리본의 각도를 수동으로 지정해둠.
2. 각 부모 본(Spine 혹은 Pelvis)과 수족 본(팔, 다리)의 쿼터니언에서 방향 벡터를 추출
3. 두 방향 벡터를 dot()계산하여 팔 다리가 어디로 뻗어있는지를 검출 (예:팔이 위로 뻗어있으면 up방향의 dot이 1, forward방향의 dot이 0이 됨)
4. dot()과 맞춤 수식의 출력값을 기준값으로 결정
5. 기준값에 따라 세컨더리 본을 미리 저장해둔 각도로 회전

<details>
	<summary>파이썬 dot검출 스크립트 소스코드(클릭 시 펼쳐집니다)</summary>
	<div markdown="1">
	
```
import bpy
import mathutils
import math
import numpy as np
import bl_math

def up_angle(traj_bone,pose_bone,parent_bone):
    rig = pose_bone.id_data
    traj_mat = (rig.matrix_world @ traj_bone.matrix).inverted()
    parent_mat = traj_mat @ parent_bone.matrix
    bone_mat = (traj_mat @ pose_bone.matrix).inverted()
    mat = parent_mat * bone_mat
    vec = mat.col[2]
    angle = math.degrees(math.asin(vec[1]))*-1
    return angle

def forward_angle(traj_bone,pose_bone,parent_bone):
    rig = pose_bone.id_data
    traj_mat = (rig.matrix_world @ traj_bone.matrix).inverted()
    parent_mat = traj_mat @ parent_bone.matrix
    bone_mat = traj_mat @ pose_bone.matrix	#(traj_mat @ pose_bone.matrix).inverted()
    parent_quat = parent_mat.to_quaternion()
    bone_quat = bone_mat.to_quaternion()
    forward = mathutils.Vector((0.0, 0.0,-1.0))
    up = mathutils.Vector((0.0,1.0,0.0))
    forward_dot = (parent_quat @ forward).normalized().dot((bone_quat @ up).normalized())
    angle = math.degrees(math.asin(forward_dot))
    return angle

def back_angle(traj_bone,pose_bone,parent_bone):
    rig = pose_bone.id_data
    traj_mat = (rig.matrix_world @ traj_bone.matrix).inverted()
    parent_mat = traj_mat @ parent_bone.matrix
    bone_mat = traj_mat @ pose_bone.matrix#(traj_mat @ pose_bone.matrix).inverted()
    parent_quat = parent_mat.to_quaternion()
    bone_quat = bone_mat.to_quaternion()
    back = mathutils.Vector((0.0, 0.0,1.0))
    up = mathutils.Vector((0.0,1.0,0.0))
    back_dot = (parent_quat @ back).normalized().dot((bone_quat @ up).normalized())
    angle = math.degrees(math.asin(back_dot))
    return angle

def left_angle(traj_bone,pose_bone,parent_bone):
    rig = pose_bone.id_data
    traj_mat = (rig.matrix_world @ traj_bone.matrix).inverted()
    parent_mat = traj_mat @ parent_bone.matrix
    bone_mat = traj_mat @ pose_bone.matrix
    parent_quat = parent_mat.to_quaternion()
    bone_quat = bone_mat.to_quaternion()
    left = mathutils.Vector((1.0,0.0,0.0))
    forward = mathutils.Vector((0.0,-1.0,0.0))
    right_dot = (parent_quat @ left).normalized().dot((bone_quat @ forward).normalized())
    angle = math.degrees(math.asin(right_dot))#right_vec.dot(mat.to_quaternion()@ mathutils.Vector((1.0,0.0,0.0)))
    return angle 
```
	
</div>
</details>




동일한 기능을 마야에서 구현한 노드입니다.

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/maya node.png?raw=true">


　
　
## 게임엔진에서 세컨더리 본 제어

-이러한 세컨더리 본은 맥스, 블렌더 등의 툴에서 키 값을 구워와야 하며, 유니티에서 움직이면 작동하지 않음.

-애니메이션 키값을 맥스에서 구워오면 문제가 되지 않음.

-그러나 애니메이터가 맥스에서 애니메이션을 잡는게 아니라, 유니티 엔진 에서 팔다리본의 움직임을 제어해야 할 경우에 세컨더리가 작동하지 않는 문제가 생김.

ex) 모션캡쳐 장비에서 캐릭터의 움직임을 받아올 경우

ex1) 계단 높낮이를 탐지하는 다리 IK

ex2) 레그돌

<img src ="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/not%20fuctional%20secondary.gif?raw=true">

-그래서 유니티에서도 세컨더리 본이 블렌더에서 처럼 작동할 수 있도록 유니티 스크립트를 구현


매트릭스를 직접 계산하여 가상 부모 설정[(FlexibleTransform)](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Matrix/FlexibleTransformController.cs)


<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/flexible%20transform.gif?raw=true">


[Secondary Bone Controller](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Rigging/SecondaryBones/SecondaryBoneController.cs/) 시연

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/unity%20secondary%20bone.gif?raw=true">
　


# 사용자정의 URP

<img src="https://github.com/0xinfinite/0xinfinite.github.io/raw/master/img/Deferred-NPR.gif?raw=true">

URP 14.0.8 패키지를 포크하여 사용자정의 파이프라인을 제작하고 있습니다. Unity 2022.3.2f1에서 구동됩니다.

[https://github.com/0xinfinite/Unity-Universal-RP](https://github.com/0xinfinite/Unity-Universal-RP) 에서 계속 업데이트 중입니다.
 

## 2D 머리카락 그림자

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/2d-hair-shadow.png?raw=true">

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/2d-hair-shadow-move.gif?raw=true">

일반 직선광으로는 재현하기 힘든 2D 머리카락 그림자를 전용 렌더러피쳐로 구현


## 안드로이드 테스트

<iframe width="560" height="315" src="https://www.youtube.com/embed/kwWVc1ryGLs?si=qsOO6tYRzaSjJ8Yr" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

테스트 기기 : 삼성 Galaxy S8+

적용 설정 : 디퍼드 렌더링, Warp Texture 셰이딩, 커스텀 셰도우 2개(전신, 2D 머리카락)

기본 설정인 채로 두어 계단현상이 있는 유니티 기본 직선광 그림자와 캐릭터에 해상도를 집중한 사용자정의 그림자를 비교할 수 있도록 다른 각도로 배치하였습니다.
 
 
# 모델링

<img src="https://imas.gamedbs.jp/cgss/images/i0kZ1jrqafsqaIn0rToUR23L0c0xW-X-dD-bzZ0hFWs.jpg">

<img src="https://dere-ken.com/wp-content/uploads/2022/03/IMG_3707.png">

예제로 쓰인 캐릭터의 원본 모델링 입니다.

<img src="https://media.discordapp.net/attachments/1043064605972381697/1140232877238407168/image0.png?ex=6534fb9e&is=6522869e&hm=121a5b4e55f3ce587c03733358da5239461758f56d8803ed16e64c3aca8788c2&=&width=330&height=586">

지인이 리터칭해 준 디자인 입니다. 치마를 배까지 올려입는 의상으로 리메이크 해보고 싶어서 부탁드렸습니다.

<iframe width="338" height="600" src="https://www.youtube.com/embed/JqNOw2amiV0?si=dryFPKsyp-Tpg_dZ&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

리터칭 이미지를 직접 모델링 한 뒤, 리깅과 애니메이션까지 진행한 작업물입니다

유니티에 올린 모습입니다.

모델링은 [https://twitter.com/Mootonashi/](https://twitter.com/Mootonashi/) 혹은 [유튜브채널](https://www.youtube.com/channel/UCa1IDNZciAUD-EPeFb5yVnQ)에서 감상하실 수 있습니다 ;)



　
　
# 언리얼 엔진

<img src="https://github.com/0xinfinite/0xinfinite.github.io/blob/master/img/unreal.png?raw=true">

최근 언리얼5를 이용하여 제작한 스크린 사격장 게임 "마스터 헌터"프로젝트에 참여했습니다.

저는 게임로직, 동물로밍AI, 전용총기컨트롤러와의 UDP통신, 서버통신연결 작업을 맡았습니다.



봐주셔서 감사합니다!

