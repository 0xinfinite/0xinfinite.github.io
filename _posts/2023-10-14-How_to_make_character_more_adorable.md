---
layout: post
title: 더 나은 방식으로 서브컬쳐 3D모델 렌더하기
---

안녕하세요, 이 글에서는 소위 '카툰 렌더링'으로 대표되는 서브컬쳐를 타겟으로 한 3D모델을 렌더하기 위해 어떤 기술을 사용했는지를 기술하고자 합니다.
이 글은 유니티 셰이더 및 스크립트 뿐만 아니라, 3D 모델 작성부터 시작해서 리깅 등 더 나은 룩스와 작업효율화를 위해 어떤 결정을 했는지 차례대로 기술하고 있습니다.
프로젝트는 깃허브에 커밋하고 있으므로 같이 보시면 좋습니다만 소제목이 하이라이트 되어있으면 그걸 누르셔도 관련된 스크립트를 열람하실 수 있습니다. (https://github.com/0xinfinite/ProjectProxy)

## 만들고자 하는 캐릭터

<iframe width="800" height="514" src="https://imas.gamedbs.jp/cgss/images/i0kZ1jrqafsqaIn0rToUR23L0c0xW-X-dD-bzZ0hFWs.jpg"></iframe>

저는 아이돌마스터 신데렐라 걸즈의 등장 캐릭터 "모치다 아리사"를 3D모델로 작성하여 렌더하는 것을 목표로 삼았습니다.

<iframe width="600" height="532" src="https://dere-ken.com/wp-content/uploads/2022/03/IMG_3707.png"></iframe>

이 캐릭터를 가장 최애하는 입장에서, 최근의 3D모델은 요즘 인기캐릭터의 유행 공식을 '일부러' 선택하지 않은 것 같아 유감스러웠습니다. 사이게임즈의 간판IP 우마무스메의 교복이나 같은 IP 인기캐릭터의 교복 테마 3D모델 의상에서 치마를 올려 입는 '하이웨이스트'로 일컫는 스타일을 채택한 데에 비해, 해당 모델의 의상은 상의가 너무 길어 다리가 짧아 보입니다.
<iframe width="337" height="600" src="https://media.discordapp.net/attachments/1043064605972381697/1140232877238407168/image0.png?ex=6534fb9e&is=6522869e&hm=121a5b4e55f3ce587c03733358da5239461758f56d8803ed16e64c3aca8788c2&=&width=330&height=586"></iframe>

그러므로 저는 기존의 디자인에서 '하이웨이스트'를 반영하여 개선한 디자인의 3D모델을 만들어 유니티 상에 렌더하고자 했습니다.(위의 이미지는 지인에게 디자인 조언을 들으며 받은 그림입니다)

## 결과물

모델링은 https://twitter.com/Mootonashi/ 혹은 [유튜브채널](https://www.youtube.com/channel/UCa1IDNZciAUD-EPeFb5yVnQ)에서 감상하실 수 있습니다 ;)

<iframe width="338" height="600" src="https://www.youtube.com/embed/UqCAhVqLBXc?si=rnOf3dps_wlWSkei&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# 블렌더 모델링

저는 이전에 3ds max와 Maya를 이용하여 각각 캐릭터 모델을 작성해본 적이 있었습니다만, 이번 모델링에서는 Blender를 이용하기로 했습니다.

Blender가 Maya에 비해 우수한 점은 Modifier제에 있다고 생각합니다.

<iframe width="630" height="390" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBuH7An69-YXSPQdHfF5S1mNKGV2xyXeV5XNJrtpKFgO_2dk8nJeN2d0lgFUqjEAnaPt6hsVtk8EUZ9Thdhb4fCsrcg6w=w1279-h832"></iframe>

Maya는 노드 기반으로 오브젝트가 이루어져있으며, 순차적으로 진행되는 작업은 히스토리에 의해 기록됩니다. 문제는, 히스토리를 무시해버리면 모델이 깨지거나 프로그램이 크래시 될 확률이 극도로 증가한다는 점입니다. 예를 들어, 이미 리깅이 진행된 모델에서 엣지를 더 생성하고 싶다던가 하면 현재의 모델을 수정하는 것이 아닌, Mesh를 복제한 새 오브젝트를 만들어 스키닝을 옮겨와야 하는 번거로움이 있습니다.

즉, Maya에서의 작업은 모델링->UV작업->스키닝->리깅 순서대로 강제되는 면이 있으며, 이를 역행하면 예기치못한 오류가 발생할 수 있습니다.

<iframe width="290" height="410" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBvB6C2me6fHyjPpLkVpZ4ykkXOB0cRoyaLnN4LCHmI77gt08cawnEJX5WRWruC5OlJs2ehQnpDDdjmETSDx4afcfIEJA=w1279-h832"></iframe>

하지만 Blender는 3ds max와 같이 Modifier를 기반으로 모델을 변형하기 때문에, **이미 리깅이 진행된 모델을 수정하는 것이 자유롭다**는 장점이 있습니다. 이는, 처음부터 완벽한 모델을 만드는 것보다 점진적으로 모델을 개선하는 작업방식에서는 Maya보다 Blender 쪽이 우수함을 의미합니다.

다만, Maya에서는 처음부터 있었던 Vertex Crease가 블렌더에서는 3.1에서야 추가되는 등, 원하는 Shape를 만드는 데에 있어선 Maya가 다소 우수하다고 생각되는 면이 있습니다.

## 세컨더리 본을 위한 Python 스크립트

Blender에서의 빠른 작업을 위해 Auto Rig Pro의 리그를 사용하여 기본 리그를 적용하였습니다. Auto Rig Pro는 트위스트 본을 지원하지만, 팔꿈치와 무릎 본은 직접 회전시켜줘야하는 불편함이 있고, 어깨와 엉덩이, 겨드랑이 등의 Secondary Bone은 직접 구현할 필요가 있었습니다.

어깨와 겨드랑이, 엉덩이는 수족이 어디에 위치하느냐에 따라 형태가 달라지기 때문에, **수족이 몸통을 기준으로 어디로 뻗어있는지**를 검출할 수 있어야 합니다. 이는 오일러 각으로는 판단할 수 없고 내부에 방향 벡터값이 있는 쿼터니언을 이용해야합니다.

아래의 코드는 몸통본(parent_bone)을 부모로 하는 매트릭스 공간에서 수족본(pose_bone)의 각 전면방향벡터가 어디를 향하고 있는지 검출하는 함수입니다. 출력된 각 값을 블렌더 리그 오브젝트의 Custom Property에 적용 후, 각 세컨더리 본에서 해당 값을 드라이버로 하는 키를 주어 어깨, 엉덩이, 겨드랑이의 움직임을 구현하였습니다.

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

이하의 영상은 세컨더리 본 움직임을 시연하는 블렌더 영상입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/5eD90udMwHE?si=SCs7uPGA7E3K1BY9" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


# 셰이딩

유니티 엔진에서 더욱 나은 퍼포먼스를 위해 프로젝트의 렌더 파이프라인은 URP를 선택했습니다.

## [StylizeLit](https://github.com/0xinfinite/ProjectProxy/tree/main/Assets/Shaders/Visuals/Stylish)

URP 파이프라인에서 전달하는 정보를 최대한 수용하면서 원하는 아트스타일의 셰이딩을 구현하려면 빛을 자유자재로 다룰 수 있어야 했습니다. 즉, 빛 데이터 자체에 접근할 필요가 있었고, Shader Graph만으로는 한계를 느껴, 기존 URP 셰이더를 복제하여 수정해서 사용하기로 했습니다.

기존 URP 셰이더에서 복제하여 수정한 셰이더는 다음과 같습니다.

1. Lit -> [StylishLit](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/Stylish/StylishLit.shader)
2. ForwardPass -> [StylishLitForwardPass](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/Stylish/StylishLitForwardPass.hlsl)
3. LitInput -> [StylishLitInput](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/Stylish/StylishLitInput.hlsl)
4. ../ShaderLibrary/SurfaceData -> [StylizedSurfaceData](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/ShaderLibrary/StylizedSurfaceData.hlsl)
5. ../ShaderLibrary/SurfaceInput -> [StylizedSurfaceInput](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/ShaderLibrary/StylizedSurfaceInput.hlsl)
6. ../ShaderLibrary/Lighting -> [StylizedLighting](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/ShaderLibrary/StylizedLighting.hlsl)
7. ../ShaderLibrary/RealtimeLights -> [StylizedRealtimeLights](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/ShaderLibrary/StylizedRealtimeLights.hlsl)
8. ../ShaderLibrary/GlobalIllumination ->[StylizedGlobalIllumination](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Shaders/Visuals/ShaderLibrary/StylizedGlobalIllumination.hlsl)

## 다른 물체의 그림자를 받으면서, 자신의 Cast Shadow를 Receiving 하지 않기

<iframe width="600" height="353" src="https://miro.medium.com/v2/resize:fit:762/0*BfyySMuoTtj15uhG.jpg"></iframe>

카툰 렌더링의 경우, 그림자 계산은 Form shadow와 Core shadow까지만 반영하고, 다른 물체에서 오는 Cast shadow를 제외하는 경우가 대부분입니다.

<iframe width="660" height="346" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDlrfpBH0qrZo3Ro0mAkeHVDWTQlFNua-vUmf4wNXfnxjXvST_vYpAEw_SghASrCA9dR6yS86WdujmOBSc1QHDVVw_ZHQ=w1101-h832"></iframe>

이유는 매우 명확한데, 얼굴에 Cast Shadow를 허용하면, 머리카락이나 얼굴 중 고저차가 높은 부분(입술, 콧대 등)에서부터 그림자가 드리워져 매우 의도하지 않은 모습을 만들어내기 때문입니다.

<iframe width="602" height="410" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaAtgIH_qNmfFi81WNcQvA_GfkkuxsAKwWsFniWKfvw8d5yDsU20eSmVdHaHNZUABs9bujZIQcFokAKHntcudDuX2eEMJA=w1101-h832"></iframe>

하지만 컴퓨터에게는 전혀 문제가 되지 않는 현상인데, 빛의 시점에서는 머리카락과 콧날이 얼굴의 다른면보다 더 가깝기 때문에 이보다 뒤에 있는 부분에 그림자가 지는 것은 **물리적**으로 당연하기 때문입니다.

그러나 NPR 아티스트들은 이를 전혀 원하지 않기 때문에 보통은 cast shadow를 비활성화 하지만, 저는 다른 물체로부터 드리우는 그림자를 연출하기 위해, Cast Shadow와 NPR 양쪽을 모두 달성할 방법을 생각했습니다.

이는 URP 패키지의 ReatimeLights.hlsl를 변형하면 해답을 얻을 수 있는데, GetMainLight()에 shadowCastOffset float 매개변수를 추가하여,  shadowCoord의 z값에 shadowCastOffset을 가산하면 머리카락과 콧날을 제외하면서도 Cast Shadow를 얻을 수 있습니다.

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

	Light GetMainLight(InputData inputData, half4 shadowMask, AmbientOcclusionFactor aoFactor, float shadowCastOffset)
	{
	    Light light = GetMainLight(inputData.shadowCoord, inputData.positionWS, shadowMask, shadowCastOffset);

	    #if defined(_SCREEN_SPACE_OCCLUSION) && !defined(_SURFACE_TYPE_TRANSPARENT)
	    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_AMBIENT_OCCLUSION))
	    {
	        light.color *= aoFactor.directAmbientOcclusion;
	    }
	    #endif

	    return light;
	}

shadowCastOffset은 마테리얼에서 조절할 수 있도록 셰이더 Property에서 선언합니다.

        _ShadowCastOffset("Shadow Cast offset", Float) = 0.3
        _AdditionalShadowCastOffset("Additional Shadow Cast offset", Float) = 0.6
        
<iframe width="600" height="316" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaD8-DaGQWyuEtHIEkS08Z_aDZmCzQnR1Wk6FHluqDk5PB72Mz9eNDlqYPvr8HyCR-bAAe7vJXWa2SNCwd5FCdKwt75uGw=w1101-h832"></iframe>
   
이렇게 하면 메인라이트 시점에서 해당 셰이더를 적용받은 픽셀 위치가 오프셋만큼 근접하는 것이 되기 때문에 (정확히는 해당 픽셀의 그림자공간 상에서의 위치값(shadowCoord)이 변동하고, ShadowCaster패스 상의 위치는 그대로이기때문에 다른 물체에 드리워질 그림자는 변화하진 않습니다.) 셰이더는 빛의 시점에서 얼굴이 머리카락보다 앞에 있다고 판단, 머리카락의 Cast shadow를 드리우지 않게 됩니다.

<iframe width="284" height="300" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaDVBCFvA0uo4g9tqJwSBpoqZ82RExdNDab3FZF2f7QspuG_pF9u5R2ywwJdtOvFR4vRAPUm4NH_j-lQLHRCd5TVnO4jzA=w1101-h501"></iframe>

다만 이 방식에는 약점이 있는데, 그림자공간상 픽셀 위치 자체를 앞으로 당겨오는 것이기 때문에, 굉장히 얇은 물체가 가까운 거리로 근접해오면 역으로 그림자가 드리워지지 않습니다. 즉, 손을 얼굴에 가까이 하면 손의 Cast Shadow가 드리워지지 않게 됩니다.
이 때에는 상황을 봐서 _ShadowCastOffset을 0으로 변동하는 식으로 대응해야 하겠습니다.

Spot Light 등의 Additional Light의 경우, 픽셀의 월드 포지션을 빛의 방향으로 offset만큼 가산해야 합니다.

	Light GetAdditionalLight(uint i, float3 positionWS, half4 shadowMask, float shadowCastOffset)
	{
	#if USE_FORWARD_PLUS
	    int lightIndex = i;
	#else
	    int lightIndex = GetPerObjectLightIndex(i);
	#endif
	    Light light = GetAdditionalPerObjectLight(lightIndex, positionWS);

	#if USE_STRUCTURED_BUFFER_FOR_LIGHT_DATA
	    half4 occlusionProbeChannels = _AdditionalLightsBuffer[lightIndex].occlusionProbeChannels;
	#else
	    half4 occlusionProbeChannels = _AdditionalLightsOcclusionProbes[lightIndex];
	#endif
	   
	    positionWS += light.direction * shadowCastOffset;
	    light.shadowAttenuation = AdditionalLightShadow(lightIndex, positionWS, light.direction, shadowMask, occlusionProbeChannels);
	#if defined(_LIGHT_COOKIES)
	    real3 cookieColor = SampleAdditionalLightCookie(lightIndex, positionWS);
	    light.color *= cookieColor;
	#endif

	    return light;
	}
   

# 얼굴 Normals 제어하기

## 서론

<iframe width="857" height="501" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBABhvYb4gTNCyd9uYLCVMPJ_fqohhQ3JOQMcB9aCsbEh7axfsDGPFpx-h9v8NaqOxxHlFmeAKn-8VBLRzLovGtvUIpSw=w1101-h832"></iframe>

카툰렌더링을 위해 만들어진 Shape은 그대로 셰이딩을 하면 굉장한 그림자를 만들어내기 때문에 얼굴에 그림자를 연출하기로 결정했으면 Normal Editing이 필수입니다.(Shader에서 얼굴 오브젝트와 vertex의 위치를 비교해서 가상 Normal을 계산하거나, 그림자를 위한 UV를 만들어내는 등의 다른 방법도 물론 있습니다.)

<iframe width="736" height="500" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaASG_ziJlsv1wwcdCeJLLX8sxtGwxaYNeG-vjZFKA-X11bvayGIlEWd_uPoEYaXllaOWN3R7KlfLY9vsvg0ZZ6M1q5Oxw=w1101-h832"></iframe>

우선 저는 Blender에서 Data Transfer 모디파이어를 이용해서 구체의 노말을 캐릭터 얼굴로 옮겼습니다. 하지만 이것은 Blender 내부에서**만** 문제가 없는 방법이며, Unity에서는 여전히 그림자 문제를 만들어냅니다.

Blendshape를 이용하여 버텍스를 옮기면, Unity는 옮겨진 버텍스와 주변 버텍스 사이의 상대위치를 기반으로 노말을 다시 계산합니다. 이는 표정이 변화할 때 눈과 입 주변에 의도하지 않은 그림자가 생기게 합니다.

이를 해결하기 위해선 Blendshape 사용 중에도 처음의 노말 정보를 계속 보존해야 합니다.

가장 간단한 방법은, **오브젝트 노말 정보를 버텍스컬러에 미리 굽고**, 셰이더에서는 노말 대신 버텍스컬러를 이용하여 dot()계산을 하는 겁니다. 하지만 저의 경우에는 후술할 외곽선을 위해 이미 버텍스컬러를 사용하고 있으므로 이 방법을 사용할 수 없었습니다.

또 다른 방법은, UV2와 UV3 채널에 노말 정보를 구워, 셰이더에서 노말 대신 UV2와 UV3 정보로 dot()계산을 하는 겁니다. 하지만 필연적으로 UV3.v 정보에는 대응할 값이 없으므로 0으로 남겨둘 수 밖에 없어 모델 용량을 낭비하게 됩니다.

다른 방법은, Skinned Mesh Renderer를 재구축하는 것입니다.

## [Custom Skinned Mesh Renderer](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Visuals/CustomSkinning/CustomSkinnedMeshRenderer.cs/)

저는 Skinned Mesh Renderer를 커스텀 스크립트로 구현하고자 했습니다. 의도하고자 한 스크립트의 작동 과정은 이하와 같습니다.

1. 원본 Skinned mesh renderer에서 매시를 복제해온다.
2. "Skinned mesh renderer"를 비활성화하고 일반 "Mesh renderer"를 활성화한다
3. 복제된 매시를 본에 따라 변형하고, 원하는 기능(지금의 경우, 노말을 변형 전의 것으로 유지)과 함께 렌더한다.

스크립트를 구현하는 과정에서 어려움을 겪은 저는 유니티 포럼에 도움을 요청, 힌트를 얻어 각 본의 localToWorldMatrix를 바인드포즈와 곱해야한다는 사실을 알았습니다.([https://forum.unity.com/threads/having-trouble-with-reproduce-skinned-mesh-renderer-manually.1499333/](https://forum.unity.com/threads/having-trouble-with-reproduce-skinned-mesh-renderer-manually.1499333/))

그리하여 구현한 코드는 다음과 같습니다.

	using UnityEngine;

	public class CustomSkinnedMeshRenderer : MonoBehaviour
	{
	    public Mesh mesh;
	    public Transform rootBone;
	    public Transform[] bones;
	    public BoneWeight[] weights;
	    [Range(0,1)]
	    public float[] blendshapeWeights;
	    public bool syncShapeValueWithSkinned = true;

	    private MeshFilter meshFilter;
	    private Mesh deformedMesh;
	    private Matrix4x4[] bindposes;
	    private Vector3[] tangents3;

	    private SkinnedMeshRenderer origSkinned;

	    void Awake()
	    {
	        meshFilter = GetComponent<MeshFilter>();

	        if(TryGetComponent<SkinnedMeshRenderer>(out origSkinned))
	        {
	            mesh = origSkinned.sharedMesh;
	            bones = origSkinned.bones;
	            weights = mesh.boneWeights;
	            blendshapeWeights = new float[origSkinned.sharedMesh.blendShapeCount];
	            rootBone = origSkinned.rootBone;
	            bindposes = mesh.bindposes;


	            origSkinned.enabled = false;
	            
	        }

	        if (mesh != null)
	        {
	            deformedMesh = DuplicateMesh(mesh);
	            meshFilter.mesh = deformedMesh;
	            tangents3 = GetTangentsToVector3Array(mesh.tangents);
	        }

	        if (TryGetComponent<MeshRenderer>(out MeshRenderer renderer))
	        {
	            renderer.sharedMaterials = origSkinned.sharedMaterials;
	            renderer.enabled = true;
	        }
	    }

	    private Mesh DuplicateMesh(Mesh sourceMesh)
	    {
	        Mesh targetMesh = new Mesh();
	        targetMesh.name = sourceMesh.name + "_Deformed";
	        targetMesh.vertices = sourceMesh.vertices;
	        targetMesh.normals = sourceMesh.normals;
	        targetMesh.tangents = sourceMesh.tangents;
	        targetMesh.triangles = sourceMesh.triangles;
	        targetMesh.uv = sourceMesh.uv;
	        targetMesh.colors = sourceMesh.colors;
	        targetMesh.subMeshCount = sourceMesh.subMeshCount;
	        targetMesh.bindposes = sourceMesh.bindposes;

	        for (int i = 0; i < targetMesh.subMeshCount; ++i)
	        {
	            targetMesh.SetSubMesh(i, sourceMesh.GetSubMesh(i));
	        }

	        return targetMesh;
	    }

	    public Vector3[] GetTangentsToVector3Array(Vector4[] tangents)
	    {
	        Vector3[] arr = new Vector3[(int)tangents.Length];

	        for(int i =0; i < tangents.Length; ++i)
	        {
	            arr[i] = tangents[i];
	        }
	        return arr;
	    }

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
	    private Vector3 DeformNormal(Vector3 normal, BoneWeight boneWeight, int weightNum)
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
	        Vector3 boneNormal = boneMatrix.MultiplyVector(normal);
	        return boneNormal * weight;

	    }
	    void LateUpdate()
	    {

	                if (mesh == null || bones == null || weights == null)
	            return;

	        Vector3[] vertices = mesh.vertices;
	        Vector3[] deltaVertices = new Vector3[vertices.Length];
	        Vector3[] normals = mesh.normals;
	        //Vector3[] deltaNormals = new Vector3[vertices.Length];

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
	                //normals[j] += deltaNormals[j] * weight;		blendshape 변형정보를 노말에 반영하고 싶을 시 주석 해제
	            }
	        }

	        for (int i = 0; i < vertices.Length; i++)
	        {
	            Vector3 deformedVertex = Vector3.zero;
	            Vector3 deformedNormal = Vector3.zero;


	            deformedVertex += DeformVertex(vertices[i] + deltaVertices[i], weights[i], 0);
	            deformedVertex += DeformVertex(vertices[i] + deltaVertices[i], weights[i], 1);
	            deformedVertex += DeformVertex(vertices[i] + deltaVertices[i], weights[i], 2);
	            deformedVertex += DeformVertex(vertices[i] + deltaVertices[i], weights[i], 3);

	            deformedNormal += DeformNormal(normals[i]/* + deltaNormals[i]*/, weights[i], 0);
	            deformedNormal += DeformNormal(normals[i]/* + deltaNormals[i]*/, weights[i], 1);
	            deformedNormal += DeformNormal(normals[i]/* + deltaNormals[i]*/, weights[i], 2);
	            deformedNormal += DeformNormal(normals[i]/* + deltaNormals[i]*/, weights[i], 3);	//blendshape정보를 노말에 반영하고 싶을 시 + deltaNormals[i] 부분 주석 해제

	            vertices[i] = transform.InverseTransformPoint(deformedVertex);
	            normals[i] = transform.InverseTransformDirection( deformedNormal.normalized);
	        }

	        deformedMesh.vertices = vertices;
	        deformedMesh.normals = normals;

	        deformedMesh.RecalculateBounds();

	    }
	}


아래 영상은 일반 Skinned mesh renderer와 Custom skinned mesh renderer를 비교시연하는 영상입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/HRp__ruzGXQ?si=XntuWlYV4mY7lsbh" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

지금 스크립트는 매시의 모든 버텍스 위치를 for문 안에서 계산하지만, Job을 이용하면 다중 스레드 상에서 매시를 변형할 수 있을 것입니다...(작업중)

# 외곽선

## 화두

<iframe width="901" height="501" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaCelr2oP8gDh_Srzx19DUpJ69fSFofvm4Kzm8aAWtZMFpZLxF0aLiyWxBs8w7ub8Fa3s5SerRAZlLWbELB4IFJsVsyqNQ=w1101-h832"></iframe>

카툰 렌더링에서 외곽선(2pass backface outline)을 렌더할때, 보통 얼굴 안에 코 이외의 외곽선이 생기지 않는 것을 희망할겁니다.

얼굴 안에 외곽선이 생기지않게 하는 기술적으로 가장 간단한 방법은 Shape에 고저차가 생기지 않도록 수동으로 버텍스를 미세조정하는 것입니다.

하지만 모든 각도에서 바라보아도 외곽선이 생기지 않게 모든 버텍스를 일일히 조정하는 것은 시간이 오래 걸립니다. 더군다나 여러가지 표정을 만들기 위해 Blendshape를 쓴다면 블랜드쉐이프의 가짓수만큼 소요시간은 비례증가합니다.

저는 시간을 절약하기 위해 Vertex colot와 Stencil를 사용하기로 결정했습니다.

## Vertex Color와 Stencil로 외곽선 마스킹하기

우선 다음과 같이 얼굴에 버텍스컬러를 발라줍니다.

<iframe width="614" height="648" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBUGrEcRGYyGG7LJUx2Gf-eRTRnNDhsNXK_JwE9YwMXMM76QSDRCwYsLEhONfV0EbgYv_WAFi1QbKegb4yRATqvHAjucA=w1101-h832"></iframe>

외곽선 fragment shader에서는 각 버텍스 컬러의 값 이하인 픽셀은 클리핑되도록 합니다.

	    finalColor.a = OutputAlpha(finalColor.a, IsSurfaceTypeTransparent(_Surface)) * input.color.r;
	    //clip(finalColor.a);
	    outColor = finalColor;

그리고 유니티 Renderer features에서 3pass를 설정하고 스탠실을 다음처럼 처리되도록 설정합니다.

1. R채널 버텍스컬러를 가진 면은 항상 렌더한다. 얼굴에서는 코의 외곽선이 렌더된다.

2. B채널 버텍스컬러를 가진 면을 마스킹으로 삼는다.(ColorMask를 0으로 하여 최종카메라에는 렌더되지 않게 한다.)

3. G채널 버텍스컬러를 가진 면을 렌더하고 B채널 면과 겹치는 부분은 지운다. 이렇게 하면 얼굴의 외곽선은 렌더되면서 얼굴 안쪽에 생기는 외곽선은 지워진다.


Stencil로 마스킹을 하면 앞서 서술한 모든 버텍스를 모든 각도에서 보면서 미세조정 하지 않아도 빠르게 예쁜 외곽선을 그려낼 수 있습니다.
단점은 4pass를 쓴다는 점이지만... URP에서는 Renderer Features에서 특정한 레이어의 오브젝트만 선택적으로 렌더하는 옵션이 있기 때문에 얼굴 오브젝트만 마스킹 렌더가 되도록 레이어를 구분한다면 큰 부하는 없으리라 생각합니다.

다음은 일반적인 2pass외곽선(좌)과 제가 구현한 외곽선(우)을 비교한 이미지입니다.

<iframe width="800" height="800" src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBnO-gc2jVoRHzsAiLJgDRvIi59o95-24KYMRFgZPcwDTVCGkfzeHwVvCmIcZVg0xtbSshMqJ9tx6dQMQy6tNP4GcT7SQ=w1101-h832"></iframe>

  
# Unity에서 세컨더리 본 제어하기

블렌더의 컨스트레인은 유니티로 옮겨올 수 없으므로, 애니메이션으로 굽지 않으면 유니티에서 세컨더리 본의 움직임을 가져올 수 없습니다.

이는 유니티에서 직접 수족의 움직임을 제어하는 경우, 세컨더리 본을 위한 커스텀 스크립트를 만들어야함을 의미합니다.

## [Flexible Transform](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Matrix/FlexibleTransformController.cs) 

세컨더리 본 스크립트를 만들기 전에, 저는 가상의 부모를 지정할 수 있는 Flexible Transform 클래스를 만들었습니다.
가상의 부모를 지정하는 것으로, SetParent()로 부모를 변경하지 않아도 가상 부모의 움직임을 반영할 수 있는 클래스를 구현했습니다.

FlexibleTransform 클래스는 처음 생성 시, 가상의 부모를 지정하여 부모와 자식 간의 로컬위치를 내부적으로 계산하여 저장합니다.

	void Start()
	    {
	        ft = new FlexibleTransform(this.transform, tempParent);
	    }

SyncTransform() 함수를 호출하는 것으로 커스텀 스크립트가 적용된 오브젝트는 가상의 부모의 월드 매트릭스에 대해 미리 정해진 상대위치로 Transform을 동기화합니다.

    void Update()
    {
        ft.SyncTransform(tempParent.localToWorldMatrix);
    }
    
다음은 히에라키 상에서 부모로 연결되어있지 않아도 부모처럼 취급되는 한 쌍의 오브젝트를 시연하는 동영상입니다.

  <iframe width="560" height="315" src="https://www.youtube.com/embed/v-nE2aIyzr0?si=Dsv9ttr05F8y1DqT" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## [Virtual Axis Transform](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Matrix/VirtualAxisTransformController.cs)

FlexibleTransform 인스턴스는 자식 트랜스폼과 부모의 매트릭스로도 생성할 수 있습니다. 다만 FlexibleTransform에서는 자식트랜스폼과 자식의 로컬 위치, 회전, 크기값만 저장되기 때문에, 부모 매트릭스도 같이 보존하고자 Virtual Axis Transform 클래스를 만들었습니다.

## [Secondary Bone Controller](https://github.com/0xinfinite/ProjectProxy/blob/main/Assets/Scripts/Rigging/SecondaryBones/SecondaryBoneController.cs/)

어깨와 겨드랑이는 상체를 가상 부모로 삼고, 엉덩이는 하체를 가상 부모로 삼아 Virtual Axis Transform을 생성합니다.

자식이 부모 공간을 기준으로 어느 방향으로 뻗어있는지를 판단하여, 회전값을 정하고 SyncTransform()을 호출하여 회전값을 반영합니다.

다음 영상은 유니티 내부에서 팔을 제어할 때, Secondary Bone Controller에 의해 움직이는 어깨본입니다. 임시로 어깨본의 전면과 상부에 자식Cube를 삽입했습니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Vc3rkIDaVZI?si=FE-RySELTz8rCFXc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# WIP

프로젝트는 계속해서 진행, 기능 추가 중입니다!

## TODO
- Custom Skinned Mesh Renderer 다중스레드 화
- 더욱 자연스러운 유니티 내 IK, FK 컨트롤
- 픽셀 선택적 Self Cast Shadow
- 블렌더 모델링의 워크플로 개선(상호 파일 변경점 업데이트 메뉴얼 등)

