服务器端命令
python src/lerobot/scripts/server/policy_server.py \
  --host=0.0.0.0 \
  --port=8080 \
  --fps=30 \
  --inference_latency=0.033 \
  --obs_queue_timeout=2
  --policy_type=smolvla \
  --pretrained_name_or_path=Lisette1231/20251027so101smolvla2 \
  --device=cuda

我的电脑命令
python src/lerobot/scripts/server/robot_client.py \
    --server_address=10.24.3.58:8080 \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --robot.cameras="{fixed: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 25}, handeye: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 25}}" \
    --task="Grab the black tape and put it on the white box" \
    --policy_type=smolvla \
    --pretrained_name_or_path=Lisette1231/20251027so101smolvla2 \
    --policy_device=cuda \
    --actions_per_chunk=50 \
    --chunk_size_threshold=0.5 \
    --aggregate_fn_name=weighted_average \
    --debug_visualize_queue_size=True



python -m lerobot.async_inference.policy_server \

​     --host=127.0.0.1 \

​     --port=8080 \

​     --fps=30 \

​     --inference_latency=0.033 \

​     --obs_queue_timeout=1



我的电脑

python src/lerobot/async_inference/robot_client.py \

​    --robot.type=so100_follower \

​    --robot.port=/dev/tty.usbmodem58760431541 \

​    --robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 1920, height: 1080, fps: 30}}" \

​    --robot.id=black \

​    --task="dummy" \

​    --server_address=127.0.0.1:8080 \

​    --policy_type=act \

​    --pretrained_name_or_path=user/model \

​    --policy_device=mps \

​    --actions_per_chunk=50 \

​    --chunk_size_threshold=0.5 \

​    --aggregate_fn_name=weighted_average \

​    --debug_visualize_queue_size=True