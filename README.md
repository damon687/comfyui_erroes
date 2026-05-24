# comfyui_erroes

#1050ti显卡安装cuda
先进入python目录
C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps
在上面的文件目录位置cmd
然后再cmd黑色命令行窗口执行下面的命令
python.exe -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 --force-reinstall

#comfyui连线
1. 在 ComfyUI 里的标准连线方式在画布中双击，搜索并添加以下两个节点即可完成调用：Upscale Image (using Model)（使用模型放大图像节点）Load Upscale Model（加载超分模型节点）连线顺序：把 Load Upscale Model 节点的 UPSCALE_MODEL 输出端，连到 Upscale Image (using Model) 节点的 upscale_model 输入端。在 Load Upscale Model 节点的下拉菜单里，直接选择 Real-ESRGAN_x4plus_anime_6B.pth。把你分布式切分出来的低清视频帧连入 IMAGE，放大后的高清帧就会从 IMAGE 输出。
2. 模型文件应该放在哪里？既然你采用了分布式架构，为了防止 234 号老机子在启动任务时去 NAS 拖模型导致千兆局域网堵塞，请直接把这个模型文件复制到四台机器各自的本地硬盘里。存放路径：你的ComfyUI纯净版目录/models/upscale_models/文件名：通常为 Real-ESRGAN_x4plus_anime_6B.pth（如果后缀是 .ckpt 或 .bin 也可以，但官方最常见的是 .pth）。
3. 完美的防爆显存连线配置（1050Ti 守护神）当你把 加载放大模型 和 使用模型放大图像 两个节点建好并连上之后，为了彻底防止 234 号机的 4G 显存（1050Ti）爆掉（OOM），我们需要对放大节点进行分块（Tile）包装：在画布空白处右键搜索并新建一个节点：Tile Model (分块模型) 或者是分布式插件特有的 Distributed Tile 节点（取决于你安装的分布式插件版本，如果没有，可以使用原生社区的 Image Tiling 节点）。最简单且绝对不会爆显存的方法：在 使用模型放大图像 节点的前面，加入一个分块参数控制。核心参数设置：Tile Width (分块宽度)：设置为 256（1050Ti 极为舒适的尺寸）。Tile Height (分块高度)：设置为 256。Tile Overlap (重叠度)：设置为 32 或 64（防止拼接处出现裂缝/接缝）

为什么这个模型特别适合你的 1050 Ti 集群？体积小、速度极快：这个模型专门针对动漫和线条做了极致的轻量化（只有不到 20MB），加载几乎秒开。对低显存极度友好：配合我们之前在 Distributed 节点里提到的 Tile（分块渲染，设为 192 或 256），234 号机的 4G 显存跑这个模型绝对稳如老狗，一次都不会闪退。顺便兼顾真人视频：你之前提到最终要放大真人视频，虽然这个模型名字带 anime（动漫），但由于它具备极强的去噪点、去伪影、平滑色块能力，用来放大 AI 动作迁移出来的真人视频反而有奇效——它能把 1 号机跑出来的低清真人脸上的“AI 脏色块”和“闪烁噪点”像磨皮一样擦干净，非常适合混合显卡集群出片。


#如何与 1 号机联动？
234 号机双击 run_worker.bat 成功启动后，你不需要在 234 号机上做任何操作了（网页都不用开）。直接回到 1号主力机 上：打开 1 号机 ComfyUI 的 Distributed 面板。输入 234 号机对应的局域网 IP（例如 192.168.1.102:8188）。点击 Connect（连接），1 号机就能瞬间把这台纯净版机子收编进你的“算力帝国”了！
