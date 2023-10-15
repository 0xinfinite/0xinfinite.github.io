---
layout: post
title: TA 포트폴리오
---
## 목차
1. [리깅](#-리깅)

  블렌더 세컨더리본

  유니티 세컨더리본

2. [렌더링](#-렌더링)

  노멀편집

  외곽선 제어

  캐스트 쉐도우 제어

3. [모델링](#-모델링)

  모치다 아리사 작업물



# 리깅

 
## 세컨더리 본 Anim Bake를 위한 Python 스크립트

팔다리 본의 위치를 인식해서 형태가 달라지는 어깨, 겨드랑이, 엉덩이 본을 파이썬 스크립트로 구현 후 Blender에 드라이버로 적용


## 어깨본

<iframe width="560" height="315" src="https://www.youtube.com/embed/FOwt6nhyBLk?si=7SCS7nW2h7zG5Joc&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



## 겨드랑이본

<iframe width="560" height="315" src="https://www.youtube.com/embed/R0mxcBDfpeA?si=rXoZePwY3h6CMStQ&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


## 엉덩이 본

<iframe width="560" height="315" src="https://www.youtube.com/embed/3r_-p-pIkzQ?si=jLB9uV57aOFdTLkK&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



<details>
	<summary>파이썬 스크립트 소스코드</summary>
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




해당 블렌더 파이썬 스크립트를 마야에서 작동할수 있도록 마야 노드로도 구현했습니다.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaA5aS9ZyGNiigBIIqMeLGf916RA1C4wzNEsaUgHjJ4vm8B0AGULWBk1vjNdhX3vAQZB198eCgLGamEuP_Gh1ZmSHBLhqA=w1200-h876">


　
　
## 게임엔진에서 세컨더리 본 제어

-이러한 세컨더리 본은 맥스, 블렌더 등의 툴에서 키 값을 구워와야 하며, 유니티에서 움직이면 작동하지 않음.

-그래서 유니티에서도 세컨더리 본이 블렌더에서 처럼 작동할 수 있도록 유니티 스크립트를 구현

<iframe width="560" height="315" src="https://www.youtube.com/embed/weaHy87mR-A?si=nsA5aeA3v9vhAais" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

매트릭스를 직접 계산하여 가상 부모 설정[(FlexibleTransform)](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Matrix/FlexibleTransformController.cs)

<iframe width="560" height="315" src="https://www.youtube.com/embed/v-nE2aIyzr0?si=Dsv9ttr05F8y1DqT" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

[Secondary Bone Controller](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Rigging/SecondaryBones/SecondaryBoneController.cs/) 시연
<iframe width="560" height="315" src="https://www.youtube.com/embed/Vc3rkIDaVZI?si=FE-RySELTz8rCFXc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


　
　
# 렌더링
 
　
## 얼굴 Normals 제어하기

카툰렌더링에서 그림자를 만화적으로 지게 만들기 위해 노말을 편집.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDlPmmMBBMrDhl4_KtzncQrIEDZZAL6j8OG8AdzmHwA-38LkURH-21n5lLm_itRd6W1yYJ67Wq-gPl_f1vrT9z45AB6Sg=w1200-h876">
<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaAGD3N8o7IadLZcPcp1AQGqxPvnJ019IUHTGJaDjUuE8viHr0F9TRaJWXQNz2CRV6HbmvwGqDFMPqMHIlcz7ERoWTdJ6Q=w1200-h876">

노멀이 편집된 상태에서 표정 애니메이션을 주니 노멀이 다시 계산되어 망가지는 문제 발생.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaA7axUWnNAxiz3CTYkPsIZoVz7oo7KGZx5uOhu1kI99vngh9ey9eXzGhzEICVRwoslaFMUl3MvzqqtgyziyHB-R1eBG=w1200-h876">

해당 문제를 해결하기 위해서 커스텀 스크립트 구현

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDXXnxs3Sal7Ahs1lgQq_d6L9CePGPlI33QmLtAWugjoDGw5a7FFR_7O1udcnXUD1skg6tkXDkvZUgCRgV87G9NPmBBeA=w1200-h876">

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
                //normals[j] += deltaNormals[j] * weight;    // 원본 매시의 노말을 유지하기 위해 블랜드셰이프의 노말 계산을 비활성화
            }
        }
	...

엔진내장 Skinned Mesh Renderer와 Custom Skinned Mesh Renderer 비교

<iframe width="560" height="315" src="https://www.youtube.com/embed/HRp__ruzGXQ?si=XntuWlYV4mY7lsbh" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



　
　
## 얼굴 안쪽의 외곽선 마스킹

표정 변화를 위해 버텍스 애니메이션을 주니 메쉬가 접히면서 쓸모없는 외곽선을 셰이더와 스탠실로 지워보겠습니다.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaAqboqcDnysQA3KA2kdk9BnnL0-NUqWcIWPJeVtg9Kkz4jAX90x_9QfI4uxfewHmijAi-a4GNwvCUJG8ZGTu1ELUuOncQ=w1619-h439">

Vertex Color를 이용

R채널 : 항상 그려지는 외곽선 렌더(코, 얼굴 이외 외곽선)

B채널 : G채널 마스킹 전용 면(ColorMask 0, 최종 카메라에 렌더되지 않음)

G채널 : B채널에 의해 마스킹 된 외곽선 렌더(얼굴외곽)


<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBUGrEcRGYyGG7LJUx2Gf-eRTRnNDhsNXK_JwE9YwMXMM76QSDRCwYsLEhONfV0EbgYv_WAFi1QbKegb4yRATqvHAjucA=w1101-h832">

의도대로 외곽선이 마스킹 된 모습

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaA8RfmiPjAY_q8k0-4Vg6YpRgjK-rgkfUEoWSyWPgx_-ctwCqugeKDjnLWBTMuAZseS76gLoayODcWCb3-LBUbc_5qVsQ=w1200-h545">

　
　
## Cast Shadow 제어하기

얼굴에 Cast Shadow를 허용한 모습

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDlrfpBH0qrZo3Ro0mAkeHVDWTQlFNua-vUmf4wNXfnxjXvST_vYpAEw_SghASrCA9dR6yS86WdujmOBSc1QHDVVw_ZHQ=w1101-h832">

-만화적인 그림자 표현을 위하여, 코의 입체로 인하여 얼굴에 드리우는 그림자를 없애기.

-RealtimeLights.hlsl를 수정하여 shadowCoord.z값을 offset만큼 가산, 얼굴 면의 그림자공간 위치 값을 머리카락과 콧날보다 앞에 오게 함

	Light GetMainLight(float4 shadowCoord, float3 positionWS, half4 shadowMask, float shadowCastOffset)
	{
	    Light light = GetMainLight();

	    shadowCoord.z += shadowCastOffset;

	    light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, shadowMask, _MainLightOcclusionProbes);

	    #if defined(_LIGHT_COOKIES)
	        real3 cookieColor = SampleMainLightCookie(positionWS);
	        light.color *= cookieColor;
	    #endif

	    return light;
	}

얼굴 그림자를 의도대로 제어하면서 Cast Shadow 양립

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaD8-DaGQWyuEtHIEkS08Z_aDZmCzQnR1Wk6FHluqDk5PB72Mz9eNDlqYPvr8HyCR-bAAe7vJXWa2SNCwd5FCdKwt75uGw=w1101-h832">




　
　

## 머리카락보다 앞에 있는 눈썹

눈썹의 버텍스 월드위치를 카메라 방향으로 앞으로 당겨서 머리카락 위에다 놓기

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

적용 전과 적용 후 비교

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDDiPJyaxarZQZT7UKAauosepef4nGj2NLnssrMRXTiL1wUvQcOH8lLQWd_-SzL3-ByKErCuH4gv7ozwR_i1MpqRL5T=w1101-h832">


　
　
# 모델링

<img src="https://imas.gamedbs.jp/cgss/images/i0kZ1jrqafsqaIn0rToUR23L0c0xW-X-dD-bzZ0hFWs.jpg">

<img src="https://dere-ken.com/wp-content/uploads/2022/03/IMG_3707.png">

예제로 쓰인 캐릭터의 원본 모델링 입니다.

<img src="https://media.discordapp.net/attachments/1043064605972381697/1140232877238407168/image0.png?ex=6534fb9e&is=6522869e&hm=121a5b4e55f3ce587c03733358da5239461758f56d8803ed16e64c3aca8788c2&=&width=330&height=586">

지인이 리터칭해 준 디자인 입니다. 치마를 배까지 올려입는 의상으로 리메이크 해보고 싶어서 부탁드렸습니다.

<iframe width="338" height="600" src="https://www.youtube.com/embed/UqCAhVqLBXc?si=rnOf3dps_wlWSkei&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

리터칭 이미지를 보며 직접 작업한 캐릭터 모델링 입니다. 모델링은 학원에서 공부하였습니다.

유니티에 올린 모습입니다.

모델링은 [https://twitter.com/Mootonashi/](https://twitter.com/Mootonashi/) 혹은 [유튜브채널](https://www.youtube.com/channel/UCa1IDNZciAUD-EPeFb5yVnQ)에서 감상하실 수 있습니다 ;)



　
　
# 언리얼 엔진

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBi8iMovKv5Pyq0mBpHfnjJGPI0dTSft54h62qcvTJqNRz8elP9OyTHpQtRpMBqA_WZFdPmQ3u8KWdNnFs_JwvszUR-cw=w1200-h809">

최근 언리얼5를 이용하여 제작한 스크린 사격장 게임 "마스터 헌터"프로젝트에 참여했습니다.

저는 게임로직, 동물로밍AI, 전용총기컨트롤러와의 UDP통신, 서버통신연결 작업을 맡았습니다.



봐주셔서 감사합니다!

