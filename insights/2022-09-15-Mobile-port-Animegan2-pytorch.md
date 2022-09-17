---
title: 'Port model with pytorch mobile'
author: AIOZ AI
authorURL: 'https://github.com/aioz-ai'
authorImageURL: 'https://avatars.githubusercontent.com/u/53249292?s=200&v=4'
tags: [ai, ml-ops]
---
## I. Ví dụ: AnimeGanv2
### Mô tả bài toán: 
Hệ thống sử dụng mô hình GAN chuyển đổi hình ảnh face người thật thành hình ảnh mang phong cách Anime. 

![Figure 1](https://drive.google.com/uc?export=view&id=1Yxq4rpcTXmWY3_9MSk9jVU6WvpFtlshW)


### Convert weight python to weight android:
Để chuyển đổi mô hình Deep Learning để chạy trên các device mobile. Chúng ta cần thực hiện công việc convert weight theo định dạng dành cho android hoặc ios. 
```
    import torch
    from model import Generator

    model = Generator()
    model.load_state_dict(torch.load('weights/animeganv2.pt', map_location='cpu' ))
    model.eval()

    example = torch.rand(1, 3, 256, 256)
    traced_script_module = torch.jit.trace(model, example)
    
    // Weight ios
    traced_script_module = torch.jit.trace(model, example)
    traced_script_module.save("animeganv2_ios.pt")
    
    // Weight android
    traced_script_module_optimized = optimize_for_mobile(traced_script_module)
    traced_script_module_optimized._save_for_lite_interpreter("animeganv2_android.ptl")
```

Note: 
* Mô hình animeganv2 chỉ yêu cầu một input tensor (N, C, W, H).
* Pytorch mobile cho phép sử dụng input với nhiều kích thước.
* Hiện tại pytorch mobile chưa hỗ trợ trên GPU vì vậy khi convert hạn chế sử dụng lệnh .cuda(). Nếu không khi build app android sẽ có báo lỗi. 

### Inference python: 
Dưới đây là phần code inference model khi thử nghiệm trên 1 hình ảnh.
```
    import cv2
    import torch
    import numpy as np
    from model import Generator
   
    net = Generator()
    net.load_state_dict(torch.load("weight.pt", map_location="cpu" ))
    net.eval()

    frame = cv2.imread('img.png')
    frame = cv2.resize(frame, (512, 512))
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB).astype(np.float32)/ 127.5 - 1.0 

    frame = torch.from_numpy(frame).cuda()
    frame = frame.permute(2, 0, 1).unsqueeze(0)
    with torch.no_grad():
        out = net(frame, False)
    out = out.squeeze(0).permute(1, 2, 0).cpu()
    out = ((out + 1.) * 127.5).numpy().clip(0, 255).astype(np.uint8)
    out = cv2.resize(out, (512, 512))
    out = cv2.cvtColor(out, cv2.COLOR_RGB2BGR)

    cv2.imshow('Frame',out)
    cv2.waitKey(0)
```

### Inference android:
#### - Install thư viên (xử lý ở file build.gradle): 
```
    implementation 'org.pytorch:pytorch_android_lite:1.10.0'
    implementation 'org.pytorch:pytorch_android_torchvision_lite:1.10.0'
```
#### - Thiết lập hệ thống (xử lý ở file MainActivity.java)
##### 1. Input image: 
- Công việc này cho phép chuẩn bị sẳn các hình ảnh mặc định để khi build app có thể đưa ra sự lựa chọn cho người dùng.
- Hình ảnh bình thước để có thể sử dụng trong android studio (java) cần phải chuyển đổi về định dạng bitmap.
- Note: đặt hình ảnh ở thư mục: ./app/src/main/assets/
```
...
   private int mImageIndex = 0;
   private String[] mTestImages = {"test1.png", "test2.jpg", "test3.png"};
   ...
   try {
         mBitmap = BitmapFactory.decodeStream(getAssets().open(mTestImages[mImageIndex]));
       } catch (IOException e) {
            Log.e("Load image", "Error reading assets", e);
            finish();
            }
...
```
##### 2. Load weight .ptl:
- Weight sao khi được convert theo định dạng của android thì cũng được load lên theo một cách riêng biệt, khi sử dụng chúng ta sẽ dùng hàm mModule cùng thư viện LiteModuleLoader.
- Note: đặt weight ở folder:  ./app/src/main/assets/

```
...
  try {
            mModule = LiteModuleLoader.load(MainActivity.assetFilePath(getApplicationContext(), "animegan_android.ptl"));
        } catch (IOException e) {
            Log.e("Load model", "Error reading assets", e);
            finish();
        }
...
```
##### 3. Xử lý input:
- Khi tiếp nhận input, đặt tính của mô hình này sẽ càng tốt khi hình ảnh đầu vào có kích thước lớn tuy nhiên khi hình ảnh có kích thước lớn thì tốc độ xử lý của mô hình sẽ chậm đi:

```
...
    Bitmap resizedBitmap = Bitmap.createScaledBitmap(mBitmap, mInputWidth, mInputHeight, true);
```
- Model nhận input phải là tensor, đồng thời phải ở định dạng dtype = float32 về khoảng từ [0;1], tensor sẽ được định hình thành (N, C, W, H):
```
    Tensor inputTensor = TensorImageUtils.bitmapToFloat32Tensor(resizedBitmap, NO_MEAN_RGB, NO_STD_RGB);
...
```
với các thông số:
```
    static float[] NO_MEAN_RGB = new float[] {0.0f, 0.0f, 0.0f};
    static float[] NO_STD_RGB = new float[] {1.0f, 1.0f, 1.0f};
```
##### 4. Model xử lý:
- Sao khi thiết lập giá trị input xong, chung ta sẽ đưa thông tin vào model thông qua công thức
```
    IValue img = IValue.from(inputTensor);
    Tensor outputTensor = mModule.forward(img).toTensor();
```
- Đang ở định dạng IValue để có thể chuyển đổi cần sử dụng .toTensor();
- Đồng thời để chuyển đổi về FloatArray thì sử dụng thêm .getDataAsFloatArray();
```
    float[] outputs = outputTensor.getDataAsFloatArray();
```
##### 5. Hậu xử lý output:
- Để chuyển đổi FloatArray hiện tại thành hình ảnh RGB chúng ta sử dụng hàm, hàm này hỗ trợ chuyển đổi [0:1] sang [0, 255]: 
```
public static Bitmap bitmapFromRGBImageAsFloatArray(float[] data, int width, int height){

    Bitmap bmp = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);

    float max = 0;
    float min = 999999;

    for(float f: data){
        if(f > max){
            max = f;
        }
        if(f < min){
            min = f;
        }
    }

    int delta = (int)(max - min);

    for (int i = 0; i < width * height; i++) {
        int r = (int) (((data[i] + 1 )* 127.5 - min) / delta);
        int g = (int) (((data[i + width * height]  + 1 )* 127.5 - min) / delta);
        int b = (int) (((data[i + width * height * 2]  + 1 )* 127.5 - min) / delta);

        int x = i / width;
        int y = i % width;
        int color = Color.rgb(r, g, b);
        bmp.setPixel(y, x, color);

    }
    return bmp;
}
...
    Bitmap results = bitmapFromRGBImageAsFloatArray(outputs, mInputWidth, mInputHeight);
```
##### 5. Kết quả đạt được
App android cho bài toán AnimeGanv2 đạt được kết quả sau:

| 640x640  | 400x400  | 300x300  |  256x256 |
| -------- | -------- | -------- | -------- |
|    9s    |    4s    |    2s    |    2s    |

![Figure 1](https://drive.google.com/uc?export=view&id=1kgbJp-Qls1IuVFS63Lks5vGN5AvkNrjn)*<center>**Figure 1**. Image Face Anime.</center>*


---

## II. Face-moving
### Mô tả bài toán: 
Bài toán này tiếp nhận 2 input cho mô hình: 
* input 1: image face.
* input 2: image face to video. 

Mô hình kp_detector tiếp nhận input 1 và input 2 để rút trình thông tin value và jacobian:
* output 1: kp_source_value
* output 2: kp_source_jacobian
* output 3: kp_driving_value
* output 4: kp_driving_jacobian
* output 5: kp_driving_init_value ( frame video 0)
* output 6: kp_driving_init_jacobian (frame video 0)

Mô hình generator tiếp nhận 7 input bao gồm: input 1 và output 1-6. Output mô hình chuyển đổi trực tiếp về hình ảnh.
![Figure 1](https://drive.google.com/uc?export=view&id=1BVfk3lApCgE1BeQkmMHdPN3hcKgzpzQv)

### Convert weight python to weight android:
* Tương tự như bài AnimeGanv2, chúng ta phải thực hiện công việc convert weight sang định dạng android. 
* Tuy nhiên mô hình này cần 2 model để xử lý nên phải convert 2 weights. 

Phần load weight và chia tách weight:

```
import matplotlib
matplotlib.use('Agg')
import os, sys
import yaml
from argparse import ArgumentParser
import torch
from sync_batchnorm import DataParallelWithCallback
from modules.generator import OcclusionAwareGenerator
from modules.keypoint_detector import KPDetector
from torch.utils.mobile_optimizer import optimize_for_mobile

if sys.version_info[0] < 3:
    raise Exception("You must use Python 3 or higher. Recommended version is Python 3.7")

if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--config",default= 'config/vox-256.yaml', help="path to config")
    parser.add_argument("--checkpoint", default='weights/vox-cpk.pth.tar', help="path to checkpoint to restore")
    parser.add_argument("--cpu", dest="cpu", action="store_true", help="cpu mode.")
    parser.set_defaults(relative=False)
    parser.set_defaults(adapt_scale=False)

    opt = parser.parse_args()

    with open(opt.config) as f:
        config = yaml.load(f)

    generator = OcclusionAwareGenerator(**config['model_params']['generator_params'],
                                        **config['model_params']['common_params'])
    if not opt.cpu:
        generator.cuda()

    kp_detector = KPDetector(**config['model_params']['kp_detector_params'],
                             **config['model_params']['common_params'])
    if not opt.cpu:
        kp_detector.cuda()
    
    if opt.cpu:
        checkpoint = torch.load(opt.checkpoint, map_location=torch.device('cpu'))
    else:
        checkpoint = torch.load(opt.checkpoint)
 
    generator.load_state_dict(checkpoint['generator'])
    kp_detector.load_state_dict(checkpoint['kp_detector'], strict=False)
    
    if not opt.cpu:
        generator = DataParallelWithCallback(generator)
        kp_detector = DataParallelWithCallback(kp_detector)

    generator.eval()
    kp_detector.eval()
```
Convert weight của model KP_detector: 
```
...
    torch.save(kp_detector.module.state_dict(), './weights/conkp_detector-vox-cpk.pth')

    model_kp_detector = KPDetector(**config['model_params']['kp_detector_params'],
                                   **config['model_params']['common_params'])

    model_kp_detector.load_state_dict(torch.load('./weights/conkp_detector-vox-cpk.pth'), strict=False)
    model_kp_detector = model_kp_detector.eval()

    example = torch.rand(1, 3, 256, 256)
    traced_script_module = torch.jit.trace(model_kp_detector, example,  strict=False)
    traced_script_module_optimized = optimize_for_mobile(traced_script_module)
    traced_script_module_optimized._save_for_lite_interpreter("./weights/model.ptl")
```
Convert weight của model Generator:
```
...
    torch.save(generator.module.state_dict(), './weights/congenerator-vox-cpk.pth')

    model_generator = OcclusionAwareGenerator(**config['model_params']['generator_params'],
                                              **config['model_params']['common_params'])
    
    model_generator.load_state_dict(torch.load('./weights/congenerator-vox-cpk.pth'), strict=False)
    model_generator = model_generator.eval()

    kp_source_value = torch.rand(1, 10, 2)
    kp_source_jacobian = torch.rand(1, 10, 2, 2)

    kp_driving_value = torch.rand(1, 10, 2)
    kp_driving_jacobian = torch.rand(1, 10, 2, 2)

    kp_driving_initial_value = torch.rand(1, 10, 2)
    kp_driving_initial_jacobian = torch.rand(1, 10, 2, 2)

    example = torch.rand(1, 3, 256, 256)

    traced_script_module = torch.jit.trace(model_generator, (example, kp_source_value, kp_source_jacobian, kp_driving_value, kp_driving_jacobian, kp_driving_initial_value, kp_driving_initial_jacobian), strict=False)
    traced_script_module_optimized = optimize_for_mobile(traced_script_module)
    traced_script_module_optimized._save_for_lite_interpreter("./weights/model.ptl")
```

Note:
* Model Kp_detector chỉ yêu cầu 1 input còn model Generator lại yêu cầu nhiều input hơn. nên có thể sử dụng lệnh như sau: 
```
    traced_script_module = torch.jit.trace(model_generator, (example, kp_source_value, kp_source_jacobian, kp_driving_value, kp_driving_jacobian, kp_driving_initial_value, kp_driving_initial_jacobian), strict=False)
```
* Vì thư viện LiteModuleLoader của android studio chỉ hỗ trợ nhiều input là dạng Ivalue vì vậy phải tách nhỏ các input thành từng biến đọc lập. Tuy nhiên output của mô hình lại cho phép chưa nhiều nội dụng. 

### Inference python: 
Dưới đây là code inference mô hình với đầu vào là 1 hình ảnh và 1 video. 
```
import matplotlib
matplotlib.use('Agg')
import os, sys
import yaml
from argparse import ArgumentParser
from tqdm import tqdm

import imageio
import numpy as np
from skimage.transform import resize
from skimage import img_as_ubyte
import torch
from sync_batchnorm import DataParallelWithCallback

from modules.generator import OcclusionAwareGenerator
from modules.keypoint_detector import KPDetector
from animate import normalize_kp
from scipy.spatial import ConvexHull

import cv2

if sys.version_info[0] < 3:
    raise Exception("You must use Python 3 or higher. Recommended version is Python 3.7")

def load_checkpoints(config_path, checkpoint_path, cpu=False):

    with open(config_path) as f:
        config = yaml.load(f)

    generator = OcclusionAwareGenerator(**config['model_params']['generator_params'],
                                        **config['model_params']['common_params'])
    if not cpu:
        generator.cuda()

    kp_detector = KPDetector(**config['model_params']['kp_detector_params'],
                             **config['model_params']['common_params'])
    if not cpu:
        kp_detector.cuda()
    
    if cpu:
        checkpoint = torch.load(checkpoint_path, map_location=torch.device('cpu'))
    else:
        checkpoint = torch.load(checkpoint_path)
 
    generator.load_state_dict(checkpoint['generator'])
    kp_detector.load_state_dict(checkpoint['kp_detector'])
    
    if not cpu:
        generator = DataParallelWithCallback(generator)
        kp_detector = DataParallelWithCallback(kp_detector)

    generator.eval()
    kp_detector.eval()
    
    return generator, kp_detector


def make_animation(source_image, driving_video, generator, kp_detector, relative=True, adapt_movement_scale=True, cpu=False):
    with torch.no_grad():
        predictions = []
        source = torch.tensor(source_image[np.newaxis].astype(np.float32)).permute(0, 3, 1, 2)
        if not cpu:
            source = source.cuda()
        driving = torch.tensor(np.array(driving_video)[np.newaxis].astype(np.float32))
        print(driving.shape)


        kp_source = kp_detector(source)
        kp_source_value = kp_source['value']
        kp_source_jacobian  = kp_source['jacobian']

        kp_driving_initial = kp_detector(driving[:, :, 0])
        kp_driving_initial_value = kp_driving_initial['value']
        kp_driving_initial_jacobian = kp_driving_initial['jacobian']

        for frame_idx in tqdm(range(driving.shape[2])):
            driving_frame = driving[:, :, frame_idx]
            if not cpu:
                driving_frame = driving_frame.cuda()
   

            kp_driving = kp_detector(driving_frame)
            kp_driving_value = kp_driving['value']
            kp_driving_jacobian = kp_driving['jacobian']

            out = generator(source_image = source, 
                            kp_source_value=kp_source_value, kp_source_jacobian=kp_source_jacobian, 
                            kp_driving_value=kp_driving_value, kp_driving_jacobian=kp_driving_jacobian, 
                            kp_driving_init_value=kp_driving_init_value, kp_driving_init_jacobian=kp_driving_init_jacobian)


            predictions.append(np.transpose(out['prediction'].data.cpu().numpy(), [0, 2, 3, 1])[0])
    return predictions

def find_best_frame(source, driving, cpu=False):
    import face_alignment

    def normalize_kp(kp):
        kp = kp - kp.mean(axis=0, keepdims=True)
        area = ConvexHull(kp[:, :2]).volume
        area = np.sqrt(area)
        kp[:, :2] = kp[:, :2] / area
        return kp

    fa = face_alignment.FaceAlignment(face_alignment.LandmarksType._2D, flip_input=True,
                                      device='cpu' if cpu else 'cuda')
    kp_source = fa.get_landmarks(255 * source)[0]
    kp_source = normalize_kp(kp_source)
    norm  = float('inf')
    frame_num = 0
    for i, image in tqdm(enumerate(driving)):
        kp_driving = fa.get_landmarks(255 * image)[0]
        kp_driving = normalize_kp(kp_driving)
        new_norm = (np.abs(kp_source - kp_driving) ** 2).sum()
        if new_norm < norm:
            norm = new_norm
            frame_num = i
    return frame_num

if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--config", required=True, help="path to config")
    parser.add_argument("--checkpoint", default='vox-cpk.pth.tar', help="path to checkpoint to restore")

    parser.add_argument("--source_image", default='sup-mat/source.png', help="path to source image")
    parser.add_argument("--driving_video", default='sup-mat/source.png', help="path to driving video")
    parser.add_argument("--result_video", default='result.mp4', help="path to output")
 
    parser.add_argument("--relative", dest="relative", action="store_true", help="use relative or absolute keypoint coordinates")
    parser.add_argument("--adapt_scale", dest="adapt_scale", action="store_true", help="adapt movement scale based on convex hull of keypoints")

    parser.add_argument("--find_best_frame", dest="find_best_frame", action="store_true", 
                        help="Generate from the frame that is the most alligned with source. (Only for faces, requires face_aligment lib)")

    parser.add_argument("--best_frame", dest="best_frame", type=int, default=None,  
                        help="Set frame to start from.")
 
    parser.add_argument("--cpu", dest="cpu", action="store_true", help="cpu mode.")
 

    parser.set_defaults(relative=False)
    parser.set_defaults(adapt_scale=False)

    opt = parser.parse_args()

    source_image = imageio.imread(opt.source_image)
    reader = imageio.get_reader(opt.driving_video)
    fps = reader.get_meta_data()['fps']
    driving_video = []
    try:
        for im in reader:
            driving_video.append(im)
    except RuntimeError:
        pass
    reader.close()

    source_image = resize(source_image, (256, 256))[..., :3]
    driving_video = [resize(frame, (256, 256))[..., :3] for frame in driving_video]
    generator, kp_detector = load_checkpoints(config_path=opt.config, checkpoint_path=opt.checkpoint, cpu=opt.cpu)

    if opt.find_best_frame or opt.best_frame is not None:
        i = opt.best_frame if opt.best_frame is not None else find_best_frame(source_image, driving_video, cpu=opt.cpu)
        print ("Best frame: " + str(i))
        driving_forward = driving_video[i:]
        driving_backward = driving_video[:(i+1)][::-1]
        predictions_forward = make_animation(source_image, driving_forward, generator, kp_detector, relative=opt.relative, adapt_movement_scale=opt.adapt_scale, cpu=opt.cpu)
        predictions_backward = make_animation(source_image, driving_backward, generator, kp_detector, relative=opt.relative, adapt_movement_scale=opt.adapt_scale, cpu=opt.cpu)
        predictions = predictions_backward[::-1] + predictions_forward[1:]
    else:
        predictions = make_animation(source_image, driving_video, generator, kp_detector, relative=opt.relative, adapt_movement_scale=opt.adapt_scale, cpu=opt.cpu)
    imageio.mimsave(opt.result_video, [img_as_ubyte(frame) for frame in predictions], fps=fps)

```
### Inference android:
#### - Install thư viên (xử lý ở file build.gradle): 
```
    implementation 'org.pytorch:pytorch_android_lite:1.10.0'
    implementation 'org.pytorch:pytorch_android_torchvision_lite:1.10.0'
```
#### - Thiết lập hệ thống (xử lý ở file MainActivity.java)
##### 1. Input image: 
* Bài toán này yêu cầu đến 2 input đầu vào 1 là hình ảnh face và 1 video face đang chuyển động. 
* Note: 
    - đặt hình ảnh ở thư mục: ./app/src/main/assets/
    - đặt video ở thư mục: ./app/src/main/res/raw/
```
    private int mImageIndex = 0;
    private String[] mTestImages = {"test1.png", "test2.png"};
    private int mTestVideoIndex = 0;
    private final String[] mTestVideos = {"video1", "video2", "video3"};
    ...
    try {
            mBitmap = BitmapFactory.decodeStream(getAssets().open(mTestImages[mImageIndex]));
        } catch (IOException e) {
            Log.e("Load image", "Error reading assets", e);
            finish();
        }
    ...
    private Uri getMedia(String mediaName) {
        return Uri.parse("android.resource://" + getPackageName() + "/raw/" + mediaName);
    }
    ...
    mVideoUri = getMedia(mTestVideos[mTestVideoIndex]);
    ...
```
##### 2. Load weights: 
   - Model KP_detector: 
```
    if (mModule_kp == null) {
            try {
                mModule_kp = LiteModuleLoader.load(MainActivity.assetFilePath(getApplicationContext(), "kp_detector.ptl"));
            } catch (IOException e) {
                Log.e("Load weight kp detector", "Error reading assets", e);
                finish();
            }
        }
```
   - Model Generator: 
```
    if (mModule_gan == null) {
            try {
                mModule_gan = LiteModuleLoader.load(MainActivity.assetFilePath(getApplicationContext(), "generator_final_v2.ptl"));
            } catch (IOException e) {
                Log.e("Load weight Generator", "Error reading assets", e);
                finish();
            }
        }
```
##### 3. Function model:
Để dễ dàng hỗ trợ khi code hệ thống, chúng ta sẽ viết các hàm riêng cho model kp_detector và generator. Các bước như resize, convert bitmaptotensor và run model sẽ tương tự như bài toán animegan.

* Model Kp_detector: 
     - Input: bitmap
     - Output: Ivalue 
```
    ...
    public IValue model_kp_detector(Bitmap bitmap){
        Bitmap resizedBitmap_kp = Bitmap.createScaledBitmap(bitmap,
                                                            PrePostProcessor.mInputWidth,
                                                            PrePostProcessor.mInputHeight,
                                                        true);
        Tensor inputTensor_img = TensorImageUtils.bitmapToFloat32Tensor(resizedBitmap_kp,
                                                                        PrePostProcessor.NO_MEAN_RGB,
                                                                        PrePostProcessor.NO_STD_RGB);

        if (mModule_kp == null) {
            try {
                mModule_kp = LiteModuleLoader.load(MainActivity.assetFilePath(getApplicationContext(), "kp_detector.ptl"));
            } catch (IOException e) {
                Log.e("Load weight kp detector", "Error reading assets", e);
                finish();
            }
        }
        final IValue output_kp = mModule_kp.forward(IValue.from(inputTensor_img));

        return output_kp;
    }
    ...
```

- Model Generator: 
     - Input[1]: bitmap
     - Input[2-3]: Ivalue kp_source (value, jacobian)
     - Input[4-5]: Ivalue kp_driving (value, jacobian)
     - Input[6-7]: Ivalue kp_driving_initial (value, jacobian)
     - Output: bitmap
```
    ...
    public Bitmap model_generator(Bitmap mBitmap,
                                  IValue kp_source,
                                  IValue kp_driving,
                                  IValue kp_driving_initial) {
        Bitmap resizedBitmap_kp = Bitmap.createScaledBitmap(mBitmap,
                                                            PrePostProcessor.mInputWidth,
                                                            PrePostProcessor.mInputHeight,
                                                        true);
        Tensor source_image = TensorImageUtils.bitmapToFloat32Tensor(resizedBitmap_kp,
                                                                    PrePostProcessor.NO_MEAN_RGB,
                                                                    PrePostProcessor.NO_STD_RGB);

        Tensor value_kp_source = kp_source.toDictStringKey().get("value").toTensor();
        Tensor jacobian_kp_source = kp_source.toDictStringKey().get("jacobian").toTensor();

        Tensor value_kp_driving = kp_driving.toDictStringKey().get("value").toTensor();
        Tensor jacobian_kp_driving = kp_driving.toDictStringKey().get("jacobian").toTensor();

        Tensor value_kp_driving_initial = kp_driving_initial.toDictStringKey().get("value").toTensor();
        Tensor jacobian_kp_driving_initial = kp_driving_initial.toDictStringKey().get("jacobian").toTensor();

        if (mModule_gan == null) {
            try {
                mModule_gan = LiteModuleLoader.load(MainActivity.assetFilePath(getApplicationContext(), "generator_final_v2.ptl"));
            } catch (IOException e) {
                Log.e("Object Detection", "Error reading assets", e);
                finish();
            }
        }

        final IValue output_gan = mModule_gan.forward(
                IValue.from(source_image),
                IValue.from(value_kp_source),
                IValue.from(jacobian_kp_source),
                IValue.from(value_kp_driving),
                IValue.from(jacobian_kp_driving),
                IValue.from(value_kp_driving_initial),
                IValue.from(jacobian_kp_driving_initial)
        );
        float[] outputs = output_gan.toTensor().getDataAsFloatArray();

        Bitmap results = bitmapFromRGBImageAsFloatArray(outputs , PrePostProcessor.mInputWidth, PrePostProcessor.mInputHeight);

        return results;
    }
    ...
```
##### 4. Xuất thông tin từ video:
Bài tóan này yêu cầu thông tin hình ảnh lấy ra từ video vì vậy phải sử dụng thư viện media để lấy thông tin: 
```
    ...
    mVideoUri = getMedia(mTestVideos[0]);
    MediaMetadataRetriever mmr = new MediaMetadataRetriever();
    mmr.setDataSource(this.getApplicationContext(), mVideoUri);
    String stringDuration = mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION);
    double durationMs = Double.parseDouble(stringDuration);

    int durationTo = (int) Math.ceil(durationMs / 50);
    for( i = 0; i < durationTo; i ++){
        int from = i * 100;
        int to = (i + 1) * 100;
        if (i == durationTo - 1)
            to = (int) Math.ceil(durationMs) - (i * 100);
        long timeUs = 100 * (from + (int) ((to - from) * i));
        Bitmap bitmap = mmr.getFrameAtTime(timeUs, MediaMetadataRetriever.OPTION_CLOSEST);
    ...
```
##### 5. Kết quả đặt được:
* Xét mô hình xử lý 1 frame ảnh trong video sẽ tiêu tổn mất 8s. 
![Figure 1](https://drive.google.com/uc?export=view&id=1TegHwqVVEuN5wUL-6F3ZtTYPI5Gl01Qm)

---
## III. Upscale 
### Mô tả bài toán: 
Bài toán sử dụng input là 1 hình ảnh face bị mờ, không rõ nét khi sử dụng mô hình sẽ qua hai model mtcnn và gpen để làm cho hình ảnh được nét hơn. 
![Figure 1](https://drive.google.com/uc?export=view&id=1VMOCJENU9jzo3kri1C3AvSKYKNhWSfuh)

### Install library:
* opencv: 
```
    https://www.youtube.com/watch?v=psoeNfFAKL8
```
* torchvision_ops:
```
    https://github.com/pytorch/vision/pull/2897
```
### Inference python: 
Dưới đây là phần code inference model với 1 hình ảnh.
```
    import cv2
    import torch
    import torchvision
    import numpy as np
    import IPython.display as ipd
    from PIL import Image

    im_pth = "tmp_of.jpg"
    img = cv2.cvtColor(cv2.imread(im_pth), cv2.COLOR_BGR2RGB)

    mtcnn = torch.jit.load("task_2/mtcnn_end2endNHWC.pt")
    gpen = torch.jit.load("task_2/Gpen512_end2endNHWCVer2.pt")

    ref512 = torch.tensor(
        [[202.04068428, 242.88396346],
         [309.43024555, 242.28998092],
         [256.07679966, 303.95917038],
         [211.95977493, 366.82819474],
         [300.89112491, 366.33630952]]).numpy()

    img_t = torch.from_numpy(img).unsqueeze(0)
    with torch.no_grad():
      boxe, landmark = mtcnn(img_t)  

    # Cv2 process
    src_pts = landmark[0][0].numpy()

    tfm = cv2.getAffineTransform(src_pts[0:3], ref512[0:3])
    tfm_inv = cv2.getAffineTransform(ref512[0:3], src_pts[0:3])
    of = cv2.warpAffine(img, tfm, (512, 512), flags=3)

    gpen_img_t = torch.from_numpy(of).unsqueeze(0)

    with torch.no_grad():
      ef = gpen(gpen_img_t)  

    # CV2 process
    ef = ef[0].numpy()
    
    tmp_mask = cv2.warpAffine(mask, tfm_inv, (512, 512), flags=3)
    gpen_im = cv2.warpAffine(ef, tfm_inv, (512, 512), flags=3)
    out_k = cv2.convertScaleAbs(img  - img * tmp_mask + gpen_im * tmp_mask)
    end = Image.fromarray(out_k)
    ipd.display(end)
```
### Inference android:
#### - Install thư viên (xử lý ở file build.gradle): 

```
    implementation 'org.pytorch:pytorch_android_lite:1.10.0'
    implementation 'org.pytorch:pytorch_android_torchvision_lite:1.10.0'
    implementation 'org.pytorch:torchvision_ops:0.10.0'
```
#### - Thiết lập hệ thống (xử lý ở file MainActivity.java):
##### 1. Input image: 
Tương tự như các bài toán khác, hình ảnh được cài đặt mặt định để thuận tiện cho việc lựa chọn của người dùng.
```
    private int mImageIndex = 0;
    private String[] mTestImages = {"test1.jpg", "test2.jpg", "test3.png"};
    ...
    try {
            mBitmap = BitmapFactory.decodeStream(getAssets().open(mTestImages[mImageIndex]));
        } catch (IOException e) {
            Log.e("Load image", "Error reading assets", e);
            finish();
        }
    ...
```
##### 2. Load weight:
Bài toán này cũng yêu cầu 2 model riêng biệt để xử lý các công việc. 
```
    ...
    try {
            mModule_mtcnn = PyTorchAndroid.loadModuleFromAsset(getAssets(), "mtcnn_end2endNCHWLandmarkLite.ptl");

            mModule_Gpen = PyTorchAndroid.loadModuleFromAsset(getAssets(), "Gpen512_end2endNCHWInOut_2Out.ptl");

        } catch (IOException e) {
            Log.e("Load weight", "Error reading assets", e);
            finish();
        }
```
##### 3. Xử lý input và output của model:
* Model Mtcnn:
    - input: Tensor([1, 3, 512, 512], dtype = uint8)
    - output: Tensor([1, 5, 2], dtype = float32)
```
    ...
    Tensor inputTensor = TensorImageUtils.bitmapToFloat32Tensor(mBitmap, NO_MEAN_RGB_255, NO_STD_RGB_255);

    IValue outputmtcnn = mModule_mtcnn.forward(IValue.from(inputTensor));
    float landmark_para[] = outputmtcnn.toTensor().getDataAsFloatArray();
    ...
```
* Model Gpen:
     - input: Tensor([1, 3, 512, 512], dtype = uint8)
     - output[0]: Tensor([1, 3, 512, 512], dtype = float32)
     - output[1]: Tensor([512, 512, 3], dtype = float32)
```
    ...
    Tensor gpen_img_t = TensorImageUtils.bitmapToFloat32Tensor(bmp, NO_MEAN_RGB_255, NO_STD_RGB_255);

    IValue out_gpen = mModule_Gpen.forward(IValue.from(gpen_img_t));
    Tensor out_gpen0 = out_gpen.toTuple()[0].toTensor();
    Tensor out_gpen1 = out_gpen.toTuple()[1].toTensor();
    ...
```
với tham số: 
```
    public final static float[] NO_MEAN_RGB_255 = new float[] {0.0f, 0.0f, 0.0f};
    public final static float[] NO_STD_RGB_255 = new float[] {1/255.f, 1/255.f, 1/255.f};
```
##### 4. Xử lý thông tin với opencv: 
* Chuyển đổi bitmap to Mat:

Để thực hiện các phép tính bằng thư viện opencv với input là hình ảnh, chúng ta cần chuyển đổi chúng về Mat. 
Cụ thể bằng cách sử dụng hàm ***Utils.bitmapToMat***.  
```
    Mat scr_img = new Mat();
    Bitmap bmp = mBitmap.copy(Bitmap.Config.ARGB_8888, true);
    Utils.bitmapToMat(bmp, scr_img);
```
Lưu ý giá trị Mat lúc này đang ở định dạng ***CvType.8UC4***.
* Chuyển đổi Mat to Bitmap:

Ngược lại, khi thao tác trên opencv đã xong chúng ta muốn xử lý hình ảnh bằng bitmap cũng có thể sử dụng hàm ***Utils.matToBitmap*** để biến đổi. 
```
    Bitmap bmp = mBitmap.copy(Bitmap.Config.ARGB_8888, true);
    Utils.matToBitmap(scr, bmp );
```
Lưu ý khi sử dụng hàm này cần phải xác định giá trị input thuộc định dạng ***CvType.8UC1, CvType.8UC3, CvType.8UC4***.

* Chuyển đổi list array to Mat:

Để xử lý các list array thuộc các định dạng byte hoặc float trên opencv chúng ta có thể sử dụng cách sao để chuyển đổi. 
```
    Mat scr = new Mat(512, 512, CvType.CV);
    scr.put(0, 0, listarray);
```
Lưu ý cần xác định định dạng của input:
     - Nếu input là ByteArray thì nên dùng ***CvType.8UC3***.
     - Nếu input là FloatArray thì nên dùng ***CvType.32FC3***.

* Chuyển đổi Mat to list array: 

Ngược lại để chuyển đổi các biến Mat về định dạng list array nào đó có thể làm như sau: 
Nếu là FloatArray (scr phải mang CvType.CV_32FC3): 
```
    float[] FloatArray = new float[(int) src.total() * scr.channels()];
    scr.get(0, 0, FloatArray);
```   
Nếu là ByteArray (scr phải mang CvType.CV_8UC3): 

``` 
    byte[] ByteArray = new byte[(int) src.total() * scr.channels()];
    scr.get(0, 0, ByteArray);
``` 
* Chuyển đổi kênh màu: 

Một cách khác để chuyển đổi kênh màu biến Mat, có thể lợi dụng cách này để chuyển đổi từ ***CvType.8UC4*** sang ***CvType.8UC3***.
```
    Mat dst = new Mat();
    Imgproc.cvtColor(scr_img, dst, Imgproc.COLOR_RGBA2RGB);
```
Ngoài ra, còn rất nhiều lệnh chuyển đổi kênh màu khác, chúng ta có thể linh động để lựa chọn biến phù hợp nhất. 
* Chuyển đổi CvType:

Khác với cách chuyển đổi kênh màu chỉ có thể chuyển đổi CvType cùng loại như 8UC1, 8UC3, 8UC4 hoặc 32FC1, 32FC3.
Chúng ta còn có thể sử dụng cách sau để chuyển đổi CvType giửa 2 kiểu khác nhau:

Chuyển đổi 8UC3 sang 32FC3:
```
    Mat dst_img = new Mat();
    dst.convertTo(dst_img, CvType.CV_32FC3, 1/255, 0);
```
Chuyển đổi 32FC3 sang 8UC3:
```
    Mat dst_img = new Mat();
    dst.convertTo(dst_img, CvType.CV_8UC3, 255, 0);
```
* Các hàm tính toán cần dùng: 

Một số hàm tính toán trong bài toàn cần dùng: 
1. hàm getAffineTransform():
```
    Mat warpMat = Imgproc.getAffineTransform( new MatOfPoint2f(srcTri), new MatOfPoint2f(dstTri) );
```
Trong đó: input và output điều là matrix [2x3].

2. hàm warpAffine():
```
    Imgproc.warpAffine( src, warpDst, warpMat, warpDst.size() );
```
Trong đó: 
    - src: Input image
    - warp_dst: Output image
    - warp_mat: Affine transform
    - warp_dst.size(): The desired size of the output image

3. hàm convertScaleAbs():
```
    Core.convertScaleAbs(src, dst);
```
Trong đó: 
    - src: input array.
    - dst: output array.
     
4. hàm multiply():
```
    Core.multiply(src1, src2, dst);
```
Trong đó: 
    - src1: first input array.
    - src2: second input array of the same size and the same type as src1.
    - dst:  output array of the same size and type as src1.

5. hàm subtract():
```
    Core.subtract(src1, src2, dst);
```
Trong đó:
    - src1: first input array or a scalar.
    - src2: second input array or a scalar.
    - dst:  output array of the same size and the same number of channels as the input array.
     
6. hàm add():
```
    Core.add(n3, n1, n4);
```
Trong đó: 
    - src1: first input array or a scalar.
    - src2: second input array or a scalar.
    - dst: output array that has the same size and number of channels as the input array(s); the depth is defined by dtype or src1/src2.
    
###### 5. Kết quả đạt được: 
hệ thống đạt được tốc độ 11s trên 1 tấm ảnh. 
![Figure 1](https://drive.google.com/uc?export=view&id=1ZLDAfVSjFoSDhpLvtX8lthtga-XAHqgQ)
