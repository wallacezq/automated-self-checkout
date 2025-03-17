

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

   