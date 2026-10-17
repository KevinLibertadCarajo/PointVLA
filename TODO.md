# PointVLA training setup checklist

Goal: use Meta Quest 3 observations and a language instruction to position the Piper H flange centre vertically 0.15 m above the top of a named desk object, with the flange pointing downward.

Hardware: desk-corner-mounted Piper H, no gripper; Quest 3. Ubuntu 24.04, ROS 2 Jazzy, AgileX ROS driver and MoveIt. Robot movement through RViz has been demonstrated by the user. GPU, firmware and exact software revisions still need recording.

## 1. Preserve the working baseline
- [x] Install Jazzy and AgileX driver; receive six-joint feedback.
- [x] Execute movement with MoveIt/RViz (user confirmed).
- [ ] Follow [WORKING_SETUP.md](WORKING_SETUP.md) to capture the actual PC configuration.
- [ ] Record firmware, exact successful launch command and any fixes made during troubleshooting.
- [ ] Preserve source revisions, submodule revisions and local modifications.
- [ ] Record desk geometry, mounting transform, robot limits and reachable downward-facing workspace.
- [ ] Keep ML dependencies separate from the working ROS environment.
Acceptance: a fresh terminal can reproduce feedback and a small planned movement.

## 2. Quest capture application
- [ ] Enable developer mode and install Unity/Android development tools.
- [ ] Run Meta's CameraViewer and CameraToWorld examples on the headset.
- [ ] Record RGB frames, capture timestamps, corresponding camera poses, intrinsics and tracking validity.
- [ ] Add episode start/stop, task instruction and success/failure controls.
- [ ] Transfer and replay recordings on the PC.
Acceptance: a short headset recording is saved and replayable.

## 3. Spatial and temporal alignment
- [ ] Install a fixed desk reference and measure its transform to the robot base.
- [ ] Estimate headset-world-to-base alignment; document axis and unit conversions.
- [ ] Validate several measured desk points while moving the headset.
- [ ] Detect tracking loss/relocalization and invalidate alignment when necessary.
- [ ] Estimate headset/PC clock offset and drift; distinguish capture from arrival time.
- [ ] Specify acceptable synchronization and spatial errors before collecting data.
Acceptance: fixed points remain stable in robot coordinates within measured tolerances.

## 4. Synchronized recording
- [ ] Record RGB, camera pose, tracking validity and instruction.
- [ ] Record /feedback/joint_states (measured state).
- [ ] Record /control/joint_states (commanded targets).
- [ ] Record MoveIt trajectory goals, execution status and relevant transforms.
- [ ] Store episode boundaries, outcomes, object arrangements and calibration version.
- [ ] Retain raw recordings independently of converted training datasets.
- [ ] Replay video alongside synchronized joint measurements and commands.
Acceptance: motion onset and completion align across streams; missing data is flagged.

## 5. Policy interface
- [ ] Start with six absolute joint-position targets in radians, ordered joint1 through joint6.
- [ ] Choose observation/action rates based on measured stream timing.
- [ ] Define resampling, image preprocessing, action horizon and normalization.
- [ ] Decide how to include camera pose; validate model compatibility.
- [ ] Implement a robot adapter with bounded targets, stale-input rejection and execution checks.
- [ ] Match deployment interpolation/control behaviour to demonstrations.
- [ ] Validate recorded action replay in simulation before hardware.
Acceptance: observation and action schemas are explicit and consistent end to end.

## 6. Pilot demonstrations
- [ ] Define flange reference point, downward orientation and 0.15 m object-top clearance.
- [ ] Measure target height and define success tolerances.
- [ ] Record 10–20 pilot episodes using MoveIt-assisted demonstrations.
- [ ] Include images before movement and a brief final hold.
- [ ] Record commands as well as feedback; annotate actual execution versus GUI preparation.
- [ ] Include different target objects among distractors so language matters.
- [ ] Review tracking, timing, joint ordering, idle periods and final accuracy.
Acceptance: pilot episodes can be replayed and independently judged successful.

## 7. Dataset and first training run
- [ ] Select compute and record GPU/VRAM.
- [ ] Set up a separate, version-pinned ML environment.
- [ ] Convert pilot episodes to the selected LeRobot dataset schema.
- [ ] Split by whole episodes/sessions/layouts, never adjacent frames.
- [ ] Configure SmolVLA or another selected pretrained policy for the input/action schema.
- [ ] Run an integration fine-tune; inspect held-out predictions without robot execution.
Acceptance: loading, training and inference work with finite, correctly scaled six-joint outputs.

## 8. Expand and evaluate
- [ ] Expand data across object placements, arm starts and lighting.
- [ ] Initially constrain headset viewpoint; progressively add head motion and occlusion.
- [ ] Treat demonstration counts as experimental budgets, not guarantees.
- [ ] Evaluate unseen layouts, viewpoints and language targets.
- [ ] Measure target selection, horizontal error, vertical clearance, interventions and rejected actions.
- [ ] Test shadow predictions, then limited supervised closed-loop motion.
- [ ] Retain workspace/collision checks and a physical emergency stop.
- [ ] Collect corrective demonstrations based on observed failures.
Acceptance: repeatable held-out task performance with documented limitations.

## References
- https://github.com/agilexrobotics/agx_arm_ros/tree/ros2
- https://github.com/oculus-samples/Unity-PassthroughCameraApiSamples
- https://huggingface.co/docs/lerobot/smolvla

Next milestone: one synchronized Quest recording and Piper H demonstration.
