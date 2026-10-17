# Recording the working robot setup

Run these commands on the Ubuntu robot PC, not inside an ML virtual environment.
This captures diagnostics and configuration; it does not enable or move the arm.
Keep your currently successful driver/MoveIt launch running to capture live parameters.
This is an inventory plus source/configuration backup, not a full disk image or a guaranteed reinstall lockfile.

## 1. Create a local snapshot

In a new Bash terminal:

```bash
source /opt/ros/jazzy/setup.bash
source ~/agx_arm_ws/install/setup.bash
snapshot_dir="$HOME/pointvla-snapshots/$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -p "$snapshot_dir"
export snapshot_dir
```

## 2. Record OS, packages and hardware

```bash
cat /etc/os-release > "$snapshot_dir/os-release.txt"
uname -a > "$snapshot_dir/kernel.txt"
python3 --version > "$snapshot_dir/python-version.txt" 2>&1
python3 -m pip list --format=json > "$snapshot_dir/python-packages.json"
dpkg-query -W -f='${binary:Package}\t${Version}\n' > "$snapshot_dir/apt-packages.tsv"
printenv ROS_DISTRO RMW_IMPLEMENTATION ROS_DOMAIN_ID > "$snapshot_dir/ros-environment.txt"
ip -details link show can0 > "$snapshot_dir/can0.txt" 2>&1
if command -v nvidia-smi >/dev/null; then
  nvidia-smi > "$snapshot_dir/gpu.txt" 2>&1
fi
```

The apt list records installed versions; availability of those versions later is not guaranteed.
The Python list does not replace the SDK source revision backup below.

## 3. Preserve exact source and local edits

This example uses the SDK location from our installation instructions. Adjust it if needed.

```bash
for repo_dir in "$HOME/agx_arm_ws/src/agx_arm_ros" "$HOME/agilex/pyAgxArm"; do
  repo_label="$(basename "$repo_dir")"
  if git -C "$repo_dir" rev-parse --is-inside-work-tree >/dev/null 2>&1; then
    git -C "$repo_dir" rev-parse HEAD > "$snapshot_dir/$repo_label-commit.txt"
    git -C "$repo_dir" status --short > "$snapshot_dir/$repo_label-status.txt"
    git -C "$repo_dir" submodule status --recursive > "$snapshot_dir/$repo_label-submodules.txt"
    git -C "$repo_dir" diff HEAD --binary > "$snapshot_dir/$repo_label-local.patch"
  fi
done
tar -czf "$snapshot_dir/ros-workspace-source.tar.gz" -C "$HOME/agx_arm_ws" src
if [ -d "$HOME/agilex/pyAgxArm" ]; then
  tar -czf "$snapshot_dir/pyAgxArm-source.tar.gz" -C "$HOME/agilex" pyAgxArm
fi
```

The source archives include untracked files and submodule contents. The top-level patch alone does not capture edits inside submodules.
If you modified files directly under install/, also preserve them:

```bash
tar -czf "$snapshot_dir/ros-install-reference.tar.gz" -C "$HOME/agx_arm_ws" install
```

The install archive is a reference copy, not a portable replacement for rebuilding.

## 4. Capture the running ROS configuration

```bash
ros2 node list > "$snapshot_dir/nodes.txt"
ros2 topic list -t > "$snapshot_dir/topics.txt"
ros2 action list -t > "$snapshot_dir/actions.txt"

mkdir -p "$snapshot_dir/parameters"
while IFS= read -r node_name; do
  [ -n "$node_name" ] || continue
  node_file="${node_name//\//_}"
  timeout 15s ros2 param dump "$node_name" > "$snapshot_dir/parameters/$node_file.yaml" 2> "$snapshot_dir/parameters/$node_file.stderr"
done < "$snapshot_dir/nodes.txt"

timeout 10s ros2 topic echo /feedback/joint_states --once > "$snapshot_dir/joint-feedback.yaml" 2>&1
```

A timeout or error means that item was not captured; inspect the output files.
Parameter dumps can include the loaded robot description and MoveIt configuration.
The fw_version parameter may be empty when detection is automatic: record detected firmware from the startup log or vendor tooling.

## 5. Write a human-readable baseline

```bash
cat > "$snapshot_dir/BASELINE.md" <<'EOF'
# Working baseline

- Date:
- Robot: AgileX Piper H
- End effector: none
- Firmware actually detected:
- OS: Ubuntu 24.04
- ROS: Jazzy
- CAN interface / bitrate:
- Mounting and desk dimensions:
- Flange/TCP definition:
- Exact successful launch command:
  (Paste the actual command used, including every argument.)
- Local source/configuration changes:
- Fix that resolved the previous start-state mismatch:
- Test completed successfully:
- Known limitations:
- Quest OS / Unity / Meta SDK versions (when available):
EOF
nano "$snapshot_dir/BASELINE.md"
```

Also copy the startup log from the successful run and save the working RViz configuration using RViz's Save Config As option into this snapshot directory.
Record the actual successful command rather than assuming an earlier chat example is identical.

## 6. Verify and back up

```bash
tar -tzf "$snapshot_dir/ros-workspace-source.tar.gz" > "$snapshot_dir/source-archive-contents.txt"
printf 'Snapshot location: %s\n' "$snapshot_dir"
```

Check that the source archives, parameter dumps and baseline notes are populated.
Copy the snapshot to a second storage location.

PointVLA is public. Keep raw archives, logs, headset recordings and machine-specific data local initially. Review and remove private paths, credentials or incidental personal information before publishing selected baseline notes. Do not upload all snapshot files with git add . automatically.

Once reviewed, commit a concise baseline document, exact launch configuration and required source patches to PointVLA. A Git tag on PointVLA alone does not pin external dependencies; retain the recorded commits and source backups.

## Restoring later

Recreate the recorded OS/ROS environment, restore source and local modifications, install the recorded SDK revision, rebuild the workspace, and use the saved launch configuration. First validate feedback and the model; then repeat the small supervised motion test. For exact machine rollback, use a separate full-disk backup in addition to this snapshot.
