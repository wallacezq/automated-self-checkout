

# Platform with Xe driver testing

## Pre-requisite

1. Recommended OS
   - Ubuntu 24.04 (preferred)

## Installation

1. Checkout the lunar-lake-test branch 

   ```
   git clone https://github.com/wallacezq/automated-self-checkout
   cd automated-self-checkout
   git checkout lunar-lake-test
   git submodule update --remote --recursive
   ```

2. Build all the container

   ```
   cd automated-self-checkout
   make build
   make download-models
   make build-benchmark
   ```

3. Run demo (output render to screen)

   **Detection only:**

   - Yolov5s (INT8):

     ```
     DISPLAY=:0 RENDER_MODE=1 PIPELINE_SCRIPT=yolov5s.sh DEVICE_ENV=res/all-gpu.env docker compose -f src/docker-compose.yml up -d
     ```

   - Yolov11s (FP16 model):

     ```
     DISPLAY=:0 RENDER_MODE=1 PIPELINE_SCRIPT=yolov11s.sh DEVICE_ENV=res/all-gpu.env docker compose -f src/docker-compose.yml up -d
     ```

     

   **Detection + Classification (effnetb0):**

   - Yolov5s (INT8):

     ```
     DISPLAY=:0 RENDER_MODE=1 PIPELINE_SCRIPT=yolov5s_effnetb0.sh DEVICE_ENV=res/all-gpu.env docker compose -f src/docker-compose.yml up -d
     ```

   - Yolov11s (FP16):

     ``` 
     DISPLAY=:0 RENDER_MODE=1 PIPELINE_SCRIPT=yolov11s_effnetb0.sh DEVICE_ENV=res/all-gpu.env docker compose -f src/docker-compose.yml up -d
     ```

   > **Note**:
   >
   > To stop the demo, run ``` make down```

4. Run stream density benchmark

   **Detection only:**

   - Yolov5s (INT8):

     ```
     cd performance-tools/benchmark-scripts/
     PIPELINE_SCRIPT=yolov5s.sh DEVICE_ENV=res/all-gpu-va.env python benchmark.py --compose_file ../../src/docker-compose.yml --target_fps 14.95
     ```

   - Yolov11s (FP16 model):

     ```
     cd performance-tools/benchmark-scripts/
     PIPELINE_SCRIPT=yolov11s.sh DEVICE_ENV=res/all-gpu-va.env python benchmark.py --compose_file ../../src/docker-compose.yml --target_fps 14.95
     ```

     

   **Detection + Classification (effnetb0):**

   - Yolov5s (INT8):

     ```
     cd performance-tools/benchmark-scripts/
     PIPELINE_SCRIPT=yolov5s_effnetb0.sh DEVICE_ENV=res/all-gpu-va.env python benchmark.py --compose_file ../../src/docker-compose.yml --target_fps 14.95
     ```

   - Yolov11s (FP16):

     ``` 
     cd performance-tools/benchmark-scripts/
     PIPELINE_SCRIPT=yolov11s_effnetb0.sh DEVICE_ENV=res/all-gpu-va.env python benchmark.py --compose_file ../../src/docker-compose.yml --target_fps 14.95
     ```



> **Note**: 
>
> - For description of the argument (eg PIPELINE_SCRIPT, DEVICE_ENV, etc) please refer to the [documentation](https://intel-retail.github.io/documentation/use-cases/automated-self-checkout/performance.html).



5. Install [qmassa](https://github.com/ulissesf/qmassa) if you need a tool to visualize GPU utilization on Intel platform with Xe driver:

   eg. For **Lunar Lake** platform:

   ``` 
   cargo install --locked --git https://github.com/ulissesf/qmassa
   $HOME/.cargo/bin/qmassa -d 0000:00:02.0
   ```

   

## Convert Ultralytics YOLOv11 model to OpenVINO format

1. See [dlstreamer documentation](https://dlstreamer.github.io/dev_guide/yolo_models.html#yolov8-yolov9-yolov10-yolo11) for more info.
2. Create a blank python script: ``` touch convert_yolo.py``` 
3. Copy the content into the file
4. Execute the script. ``` python convert_yolo.py```. Note that this script will generate 2 new folder, namely **FP32/** and **FP16/**.

```python
from ultralytics import YOLO
import openvino, sys, shutil, os

model_name = 'yolo11s'
model_type = 'yolo_v11'
weights = model_name + '.pt'
model = YOLO(weights)
model.info()

converted_path = model.export(format='openvino')
converted_model = converted_path + '/' + model_name + '.xml'

core = openvino.Core()

ov_model = core.read_model(model=converted_model)
if model_type in ["YOLOv8-SEG", "yolo_v11_seg"]:
    ov_model.output(0).set_names({"boxes"})
    ov_model.output(1).set_names({"masks"})
ov_model.set_rt_info(model_type, ['model_info', 'model_type'])

openvino.save_model(ov_model, './FP32/' + model_name + '.xml', compress_to_fp16=False)
openvino.save_model(ov_model, './FP16/' + model_name + '.xml', compress_to_fp16=True)

shutil.rmtree(converted_path)
os.remove(f"{model_name}.pt")
```

2. Copy the generated FP32 and FP16 folder into the object_detection model directory

   ``` 
   cd automated-self-checkout
   mkdir -p models/object_detection/yolov11s
   mv FP32/ models/object_detection/yolov11s
   mv FP16/ models/object_detection/yolov11s
   ```

   