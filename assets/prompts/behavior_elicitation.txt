lv_1 = "how much force to {task} while grasping the {obj}"

lv_2 = """
Given the user instruction generate a structured physical plan.
The task is to {task} while grasping the {obj}.

Reason about the provided and implicit information in the task description to generate a structured plan for motion and forces. Think about:
- Object geometry and contact points
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)
- Force and torque requirements

[start of motion plan]
Understanding Applied Forces and Torques to Accomplish Task:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object and environment and accomplish the motion goal}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N should be exerted on the object.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N should be exerted on the object.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N should be exerted on the object.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m should be exerted on the object.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m should be exerted on the object.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m should be exerted on the object.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .

Python Code with Final Motion Plan:
```python
# succinct text description of the explicit estimated physical properties of the object, including mass, material, friction coefficients, etc.
property_description = "{{DESCRIPTION: describe succinctly the object and its properties}}"
# succinct text description of the motion plan along the wrist axes
motion_description = "{{DESCRIPTION: the object's required position change to accomplish the task}}"
# the vector (sign of direction * magnitude) of the forces and torques [x, y, z, rx, ry, rz] to accomplish the task
wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known environmental capabilities. Remember that forces in additional axes compared to the motion direction may be required to accomplish the task.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
"""

lv_3 = """
Given the user instruction, generate a structured physical plan for a robot with an appropriate end-effector interacting with the environment and object.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot end-effector, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .

Python Code with Final Motion Plan:
```python
# succinct text description of the explicit estimated physical properties of the object, including mass, material, friction coefficients, etc.
property_description = "{{DESCRIPTION: describe succinctly the object and its properties}}"
# succinct text description of the motion plan along the wrist axes
wrist_motion_description = "{{DESCRIPTION: the object's required position motion in the wrist frame to accomplish the task}}"
# the vector (sign of direction * magnitude) of the forces and torques along the wrist's [x, y, z, rx, ry, rz] axes
wrist_wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```

[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
"""

lv_4 = "how much force to {task} while grasping the {obj} in the image"

lv_5 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the visual information and semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot gripper, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object in the image.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object in the image.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object in the image.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object in the image.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .

Python Code with Final Motion Plan:
```python
# succinct text description of the explicit estimated physical properties of the object, including mass, material, friction coefficients, etc.
property_description = "{{DESCRIPTION: describe succinctly the object and its properties}}"
# succinct text description of the motion plan along the wrist axes
wrist_motion_description = "{{DESCRIPTION: the object's required position motion in the wrist frame to accomplish the task}}"
# the vector (sign of direction * magnitude) of the forces and torques along the wrist's [x, y, z, rx, ry, rz] axes
wrist_wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```

[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_6 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.

Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Mapping World Motion to Wrist Motion:
The provided workspace image confirms {{DESCRIPTION: the object and environment in the image and their properties, such as spatial relationships, color, shape, and material, and their correspondence to the requested task}}.
The labeled wrist axes correspond to the world as such: {{DESCRIPTION: describe in detail the labeled wrist frame's axes of motion and their correspondence to the motion in the depicted world, utilizing the provided World Motion Reference}}.
The blue axis represents wrist Z-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Z-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Z-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Z-axis to accomplish the task}}.
The red axis represents wrist X-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist X-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist X-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist X-axis to accomplish the task}}.
The green axis represents wrist Y-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Y-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Y-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Y-axis to accomplish the task}}.
To accomplish the task in the wrist frame, the object must be moved {{DESCRIPTION: the object's required motion in the wrist frame to accomplish the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object in the image.
Linear Y-axis:  To complete the task, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object in the image.
Linear Z-axis:  To complete the task, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object in the image.
Angular X-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Y-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Z-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object in the image.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .

Python Code with Final Motion Plan:
```python
# succinct text description of the explicit estimated physical properties of the object, including mass, material, friction coefficients, etc.
property_description = "{{DESCRIPTION: describe succinctly the object and its properties}}"
# succinct text description of the motion plan along the wrist axes
wrist_motion_description = "{{DESCRIPTION: the object's required position motion in the wrist frame to accomplish the task}}"
# the vector (sign of direction * magnitude) of motion across the wrist axes [x, y ,z]. 
wrist_motion_vector = [{{NUM}}, {{NUM}}, {{NUM}}]
# the vector (sign of direction * magnitude) of the forces and torques along the wrist's [x, y, z, rx, ry, rz] axes
wrist_wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```

[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_7 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Mapping World Motion to Wrist Motion:
The provided workspace image confirms {{DESCRIPTION: the object and environment in the image and their properties, such as spatial relationships, color, shape, and material, and their correspondence to the requested task}}.
The labeled wrist axes correspond to the world as such: {{DESCRIPTION: describe in detail the labeled wrist frame's axes of motion and their correspondence to the motion in the depicted world, utilizing the provided World Motion Reference}}.
The blue axis represents wrist Z-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Z-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Z-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Z-axis to accomplish the task}}.
The red axis represents wrist X-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist X-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist X-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist X-axis to accomplish the task}}.
The green axis represents wrist Y-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Y-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Y-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Y-axis to accomplish the task}}.
To accomplish the task in the wrist frame, the object must be moved {{DESCRIPTION: the object's required motion in the wrist frame to accomplish the task}}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the visual information and semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot gripper, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .

Python Code with Final Motion Plan:
```python
# succinct text description of the explicit estimated physical properties of the object, including mass, material, friction coefficients, etc.
property_description = "{{DESCRIPTION: describe succinctly the object and its properties}}"
# succinct text description of the motion plan along the wrist axes
wrist_motion_description = "{{DESCRIPTION: the object's required position motion in the wrist frame to accomplish the task}}"
# the vector (sign of direction * magnitude) of motion across the wrist axes [x, y ,z]. 
wrist_motion_vector = [{{NUM}}, {{NUM}}, {{NUM}}]
# the vector (sign of direction * magnitude) of the forces and torques along the wrist's [x, y, z, rx, ry, rz] axes
wrist_wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```

[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_8 = """
Given the user instruction generate a structured physical plan.
The task is to {task} while grasping the {obj}.

Reason about the provided and implicit information in the task description to generate a structured plan for motion and forces.

[start of motion plan]
Python Code with Final Motion Plan:
```python
# succinct text description of the motion plan along the wrist axes
motion_description = "{{DESCRIPTION: the object's required position change to accomplish the task}}"
# the vector (sign of direction * magnitude) of the forces and torques [x, y, z, rx, ry, rz] to accomplish the task
wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known environmental capabilities. Remember that forces in additional axes compared to the motion direction may be required to accomplish the task.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
"""

lv_9 = """
Given the user instruction and image, generate a structured physical plan for a robot end-effector interacting with the environment to accomplish.
The task is to {task} while grasping the {obj}.

Reason about the provided visual and implicit information in the task description to generate a structured plan for motion and forces.

[start of motion plan]
Python Code with Final Motion Plan:
```python
# succinct text description of the motion plan along the wrist axes
motion_description = "{{DESCRIPTION: the object's required position change to accomplish the task}}"
# the vector (sign of direction * magnitude) of the forces and torques [x, y, z, rx, ry, rz] to accomplish the task
wrench = [{{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}, {{NUM}}]
# the grasping force, which must be positive
grasp_force = {{PNUM}}
# the task duration, which must be positive
duration = {{PNUM}}
```
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known environmental capabilities. Remember that forces in additional axes compared to the motion direction may be required to accomplish the task.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
7. Make sure to provide the final python code for each requested force in a code block. Remember to fully replace the placeholder text with the actual values!
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
"""

lv_10 = """
Given the user instruction, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot gripper, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object in the image.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object in the image.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object in the image.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object in the image.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_11 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the visual information and semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot gripper, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object in the image.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object in the image.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object in the image.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object in the image.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_12 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.

Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Mapping World Motion to Wrist Motion:
The provided workspace image confirms {{DESCRIPTION: the object and environment in the image and their properties, such as spatial relationships, color, shape, and material, and their correspondence to the requested task}}.
The labeled wrist axes correspond to the world as such: {{DESCRIPTION: describe in detail the labeled wrist frame's axes of motion and their correspondence to the motion in the depicted world, utilizing the provided World Motion Reference}}.
The blue axis represents wrist Z-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Z-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Z-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Z-axis to accomplish the task}}.
The red axis represents wrist X-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist X-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist X-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist X-axis to accomplish the task}}.
The green axis represents wrist Y-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Y-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Y-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Y-axis to accomplish the task}}.
To accomplish the task in the wrist frame, the object must be moved {{DESCRIPTION: the object's required motion in the wrist frame to accomplish the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task, the robot must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N on the object in the image.
Linear Y-axis:  To complete the task, the robot must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N on the object in the image.
Linear Z-axis:  To complete the task, the robot must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N on the object in the image.
Angular X-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Y-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m on the object in the image.
Angular Z-axis: To complete the task, the robot must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m on the object in the image.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""

lv_13 = """
Given the user instruction and an image of the robot workspace, generate a structured physical plan for a robot end-effector interacting with the environment.
The task is to {task} while grasping the {obj}.

The robot is controlled using position and torque-based control, with access to contact feedback and 6D motion capabilities. 
Motions can include grasping, lifting, pushing, tapping, sliding, rotating, or any interaction with objects or surfaces.
Motion can resolve across multiple axes, so be careful to consider all axes of motion to accomplish the task.

Reason about the provided and implicit information in the images and task description to generate a structured plan for the robot's motion. Think about:
- Object geometry and contact points (from the image)
- Force/torque sensing at the wrist
- Prior knowledge of object material types and mass estimates
- Environmental knowledge (table, gravity, hinge resistance, etc.)

The robot workspace view labeled with the axes of motion relative to the wrist of the robot, placed at the point of grasping. The wrist of the robot may be oriented differently from the canonical world-axes, so this workspace view may help understand the wrist-relative motion to accomplish the task in the world.
World Motion Reference: As ground truth reference for world motion relative to the robot, the workspace camera view roughly corresponds to canonical motion in the world. 
Upward motion in the world corresponds to motion up workspace camera view image, 
Right-ward motion in the world corresponds to motion to the right of the workspace camera view image, 
And forward motion in the world corresponds to motion into the workspace camera view image.
The wrist frame axes will roughly correspond to these directions, but will be more aligned with the gripper-object relative motion.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.
Use physical reasoning to complete the following plan in a structured format. Carefully map the required motion in the world to the required motion, forces, and torques at the wrist.

[start of motion plan]
The task is to {task} while grasping the {obj}.

Mapping World Motion to Wrist Motion:
The provided workspace image confirms {{DESCRIPTION: the object and environment in the image and their properties, such as spatial relationships, color, shape, and material, and their correspondence to the requested task}}.
The labeled wrist axes correspond to the world as such: {{DESCRIPTION: describe in detail the labeled wrist frame's axes of motion and their correspondence to the motion in the depicted world, utilizing the provided World Motion Reference}}.
The blue axis represents wrist Z-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Z-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Z-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Z-axis to accomplish the task}}.
The red axis represents wrist X-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist X-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist X-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist X-axis to accomplish the task}}.
The green axis represents wrist Y-axis motion. It roughly corresponds to {{DESCRIPTION: describe the wrist Y-axis (positive and negative) motion to motion in the world with careful analysis.}}.
Based off knowledge of the task and motion, in the wrist Y-axis, the object must move {{DESCRIPTION: the object's required motion in the wrist Y-axis to accomplish the task}}.
To accomplish the task in the wrist frame, the object must be moved {{DESCRIPTION: the object's required motion in the wrist frame to accomplish the task}}.

Understanding Robot-Applied Forces and Torques to Move Object in the Wrist Frame:
To estimate the forces and torques required to accomplish {task} while grasping the {obj}, we must consider the following:
- Object Properties: {{DESCRIPTION: Think very carefully about the estimated mass, material, stiffness, friction coefficient of the object based off the visual information and semantic knowledge about the object. If object is articulated, do the same reasoning for whatever joint / degree of freedom enables motion. }}.
- Environmental Factors: {{DESCRIPTION: Think very carefully about the various environmental factors in task like gravity, surface friction, damping, hinge resistance that would interact with the object over the course of the task}}.
- The relevant object is {{DESCRIPTION: describe the object and its properties}} has mass {{NUM}} kg and, with the robot gripper, has a static friction coefficient of {{NUM}}.
- The surface of interaction is {{DESCRIPTION: describe the surface and its properties}} has a static friction coefficient of {{NUM}} with the object.
- Contact Types: {{DESCRIPTION: consideration of various contacts such as edge contact, maintaining surface contact, maintaining a pinch grasp, etc.}}.
- Motion Type: {{DESCRIPTION: consideration of forceful motion(s) involved in accomplishing task such as pushing forward while pressing down, rotating around hinge by pulling up and out, or sliding while maintaining contact}}.
- Contact Considerations: {{DESCRIPTION: explicitly consider whether additional axes of force are required to maintain contact with the object, robot, and environment and accomplish the motion goal}}.
- Motion along axes: {{DESCRIPTION: e.g., the robot exerts motion in a “linear,” “rotational,” “some combination” fashion along the wrist's [x, y, z, rx, ry, rz] axes}}.
- Task duration: {{DESCRIPTION: reasoning about the task motion, forces, and other properties to determine an approximate time duration of the task, which must be positive}}.

Physical Model (if applicable):
- Relevant quantities and estimates: {{DESCRIPTION: include any relevant quantities and estimates used in the calculations}}.
- Relevant equations: {{DESCRIPTION: include any relevant equations used in the calculations}}.
- Relevant assumptions: {{DESCRIPTION: include any relevant assumptions made in the calculations}}.
- Computations: {{DESCRIPTION: include in full detail any relevant calculations using the above information}}.
- Force/torque motion computations with object of mass {{NUM}} kg and static friction coefficient of {{NUM}} along the surface: {{DESCRIPTION: for the derived or estimated motion, compute the force required to overcome friction and achieve the task}}.

Wrist Force/Torque Motion Estimation:
Linear X-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: [positive, negative, no]}} force along the X-axis with magnitude {{PNUM}} N.
Linear Y-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: [positive, negative, no]}} force along the Y-axis with magnitude {{PNUM}} N.
Linear Z-axis:  To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: linear [positive, negative, no]}} force along the Z-axis with magnitude {{PNUM}} N.
Angular X-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the X-axis with magnitude {{PNUM}} N-m.
Angular Y-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Y-axis with magnitude {{PNUM}} N-m.
Angular Z-axis: To complete the task and based upon {{DESCRIPTION: reasoning about and estimation of task physical properties}}, the object in the image must exert {{CHOICE: angular [counterclockwise, clockwise, no]}} torque about the Z-axis with magnitude {{PNUM}} N-m.
Grasping force:  {{DESCRIPTION: estimated force range and justification based on friction, mass, resistance}}, thus {{PNUM}} to {{PNUM}} N .
[end of motion plan]

Rules:
1. Replace all {{DESCRIPTION: ...}}, {{PNUM}}, {{NUM}}, and {{CHOICE: ...}} entries with specific values or statements. For example, {{PNUM}} should be replaced with a number like 0.5. This is very important for downstream parsing!!
2. Use best physical reasoning based on known robot/environmental capabilities. Remember that the robot may have to exert forces in additional axes compared to the motion direction axes in order to maintain contacts between the object, robot, and environment.
3. Always include motion for all axes of motion, even if it's "No motion required."
4. Keep the explanation concise but physically grounded. Prioritize interpretability and reproducibility.
5. Use common sense where exact properties are ambiguous, and explain assumptions.
6. Do not include any sections outside the start/end blocks or add non-specified bullet points.
8. Do not abbreviate the prompt when generating the response. Fully reproduce the template, but filled in with your reasoning.
9. Make sure to refer to the provided correspondence in the direction guide between motion in the world frame and positive/negative motion in the respective axes.
"""