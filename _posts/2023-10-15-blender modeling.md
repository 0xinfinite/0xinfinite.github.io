---
layout: post
title: 블렌더 모델링의 장점과 세컨더리 본 스크립트
---
　

# 블렌더 모델링

저는 이전에 3ds max와 Maya를 이용하여 각각 캐릭터 모델을 작성해본 적이 있었습니다만, 이번 모델링에서는 Blender를 이용하기로 했습니다.

Blender가 Maya에 비해 우수한 점은 Modifier제에 있다고 생각합니다.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBuH7An69-YXSPQdHfF5S1mNKGV2xyXeV5XNJrtpKFgO_2dk8nJeN2d0lgFUqjEAnaPt6hsVtk8EUZ9Thdhb4fCsrcg6w=w1279-h832">

Maya는 노드 기반으로 오브젝트가 이루어져있으며, 순차적으로 진행되는 작업은 히스토리에 의해 기록됩니다. 문제는, 히스토리를 무시해버리면 모델이 깨지거나 프로그램이 크래시 될 확률이 극도로 증가한다는 점입니다. 예를 들어, 이미 리깅이 진행된 모델에서 엣지를 더 생성하고 싶다던가 하면 현재의 모델을 수정하는 것이 아닌, Mesh를 복제한 새 오브젝트를 만들어 스키닝을 옮겨와야 하는 번거로움이 있습니다.

즉, Maya에서의 작업은 모델링->UV작업->스키닝->리깅 순서대로 강제되는 면이 있으며, 이를 역행하면 예기치못한 오류가 발생할 수 있습니다.

<img src="https://lh3.googleusercontent.com/u/0/drive-viewer/AK7aPaBvB6C2me6fHyjPpLkVpZ4ykkXOB0cRoyaLnN4LCHmI77gt08cawnEJX5WRWruC5OlJs2ehQnpDDdjmETSDx4afcfIEJA=w1279-h832">

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
　
　
　
