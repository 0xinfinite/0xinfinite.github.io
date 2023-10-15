---
layout: post
title: TA 포트폴리오
---


<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBi8iMovKv5Pyq0mBpHfnjJGPI0dTSft54h62qcvTJqNRz8elP9OyTHpQtRpMBqA_WZFdPmQ3u8KWdNnFs_JwvszUR-cw=w1200-h809">

최근 언리얼5를 이용한 프로젝트에 클라이언트 프로그래머로 참여한 경험이 있습니다.

이하는 유니티 작업물입니다.


　
　
## 세컨더리 본 Anim Bake를 위한 Python 스크립트

팔다리 본의 위치를 인식해서 형태가 달라지는 어깨, 겨드랑이, 엉덩이 본을 파이썬 스크립트로 구현 후 Blender에 드라이버로 적용
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


어깨본

<iframe width="560" height="315" src="https://www.youtube.com/embed/FOwt6nhyBLk?si=7SCS7nW2h7zG5Joc&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


겨드랑이본

<iframe width="560" height="315" src="https://www.youtube.com/embed/R0mxcBDfpeA?si=rXoZePwY3h6CMStQ&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


엉덩이 본

<iframe width="560" height="315" src="https://www.youtube.com/embed/3r_-p-pIkzQ?si=jLB9uV57aOFdTLkK&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

엉덩이본을 Maya 노드 에디터로 구현한 것

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaA5aS9ZyGNiigBIIqMeLGf916RA1C4wzNEsaUgHjJ4vm8B0AGULWBk1vjNdhX3vAQZB198eCgLGamEuP_Gh1ZmSHBLhqA=w1200-h876">



　
　
## 얼굴 Normals 제어하기

Normals을 Transfer해도 Blendshape를 주면 노말이 다시 계산되어 이상한 그림자가 지게 됨



[Custom Skinned Mesh Renderer 스크립트](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Visuals/CustomSkinning/CustomSkinnedMeshRenderer.cs) 구현

	...
    private Vector3 DeformVertex(Vector3 vertex, BoneWeight boneWeight, int weightNum)
    {

        float weight = boneWeight.weight0;

        if (weight <= 0) return Vector3.zero;

        Transform bone = bones[boneWeight.boneIndex0];
        int boneIndex = boneWeight.boneIndex0;
        switch (weightNum)
        {
            default:                
                break;
            case 1:
                weight = boneWeight.weight1;
                boneIndex = boneWeight.boneIndex1;
                bone = bones[boneWeight.boneIndex1];
                break;
            case 2:
                weight = boneWeight.weight2;
                boneIndex = boneWeight.boneIndex2;
                bone = bones[boneWeight.boneIndex2];
                break;
            case 3:
                weight = boneWeight.weight3;
                boneIndex = boneWeight.boneIndex3;
                bone = bones[boneWeight.boneIndex3];
                break;
        }

        Matrix4x4 boneMatrix = bone.localToWorldMatrix * bindposes[boneIndex]; 
        Vector3 boneVertex = boneMatrix.MultiplyPoint3x4(vertex);
        return boneVertex * weight;

    }
	...

엔진내장 Skinned Mesh Renderer와 Custom Skinned Mesh Renderer 비교

<iframe width="560" height="315" src="https://www.youtube.com/embed/HRp__ruzGXQ?si=XntuWlYV4mY7lsbh" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



　
　
## 얼굴 안쪽의 외곽선 마스킹

Blendshape에 의해 얼굴에 고저차가 생겨 쓸모없는 외곽선이 생긴 모습

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBnO-gc2jVoRHzsAiLJgDRvIi59o95-24KYMRFgZPcwDTVCGkfzeHwVvCmIcZVg0xtbSshMqJ9tx6dQMQy6tNP4GcT7SQ=w1101-h832">

Vertex Color를 이용
R채널 : 항상 그려지는 외곽선 렌더(코, 얼굴 이외 외곽선)
B채널 : G채널 마스킹 전용 면(ColorMask 0, 최종 카메라에 렌더되지 않음)
G채널 : B채널에 의해 마스킹 된 외곽선 렌더(얼굴외곽)

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBUGrEcRGYyGG7LJUx2Gf-eRTRnNDhsNXK_JwE9YwMXMM76QSDRCwYsLEhONfV0EbgYv_WAFi1QbKegb4yRATqvHAjucA=w1101-h832">

의도대로 외곽선이 마스킹 된 모습

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaA8RfmiPjAY_q8k0-4Vg6YpRgjK-rgkfUEoWSyWPgx_-ctwCqugeKDjnLWBTMuAZseS76gLoayODcWCb3-LBUbc_5qVsQ=w1200-h545">

　
　
## Cast Shadow 제어하기

얼굴에 Cast Shadow를 허용한 모습

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDlrfpBH0qrZo3Ro0mAkeHVDWTQlFNua-vUmf4wNXfnxjXvST_vYpAEw_SghASrCA9dR6yS86WdujmOBSc1QHDVVw_ZHQ=w1101-h832"><

RealtimeLights.hlsl를 수정하여 shadowCoord.z값을 offset만큼 가산, 얼굴 면의 그림자공간 위치 값을 머리카락과 콧날보다 앞에 오게 함

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


　
　
## 게임엔진에서 세컨더리 본 제어

Animation Clip가 아닌 직접 수족을 제어할 경우 세컨더리 본이 작동하지 않음

<iframe width="560" height="315" src="https://www.youtube.com/embed/weaHy87mR-A?si=nsA5aeA3v9vhAais" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

매트릭스를 직접 계산하여 가상 부모 설정[(FlexibleTransform)](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Matrix/FlexibleTransformController.cs)

<iframe width="560" height="315" src="https://www.youtube.com/embed/v-nE2aIyzr0?si=Dsv9ttr05F8y1DqT" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

[Secondary Bone Controller](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Rigging/SecondaryBones/SecondaryBoneController.cs/) 시연
<iframe width="560" height="315" src="https://www.youtube.com/embed/Vc3rkIDaVZI?si=FE-RySELTz8rCFXc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


　
　

## 머리카락보다 앞에 있는 눈썹

버텍스 월드위치를 뷰 디렉션 방향으로 offset만큼 이동

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


　
　
## 3D 모델링

<img src="https://imas.gamedbs.jp/cgss/images/i0kZ1jrqafsqaIn0rToUR23L0c0xW-X-dD-bzZ0hFWs.jpg">

<img src="https://dere-ken.com/wp-content/uploads/2022/03/IMG_3707.png">

예제로 쓰인 캐릭터의 원본 모델링 입니다.

<img src="https://media.discordapp.net/attachments/1043064605972381697/1140232877238407168/image0.png?ex=6534fb9e&is=6522869e&hm=121a5b4e55f3ce587c03733358da5239461758f56d8803ed16e64c3aca8788c2&=&width=330&height=586">

지인이 리터칭해 준 디자인 입니다.

<iframe width="338" height="600" src="https://www.youtube.com/embed/UqCAhVqLBXc?si=rnOf3dps_wlWSkei&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

리터칭 이미지를 보며 직접 작업한 캐릭터 모델링 입니다. 유니티에 올린 모습입니다.

모델링은 [https://twitter.com/Mootonashi/](https://twitter.com/Mootonashi/) 혹은 [유튜브채널](https://www.youtube.com/channel/UCa1IDNZciAUD-EPeFb5yVnQ)에서 감상하실 수 있습니다 ;)

봐주셔서 감사합니다!

