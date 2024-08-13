# Sora相关技术
Sora是OpenAI推出的通用视频生成模型，可以生成1分钟高清视频，并适应不同的时长、分辨率和纵横比。Sora的成功，以一系列业界、学界的计算机视觉、自然语言处理等的技术进展作为支撑，PPDiffusers针对Sora相关的核心技术能力进行支持。

## 视频压缩网络（video compression network）
Sora采用Latent Diffusion架构，它通过一个视频压缩网络降低视觉数据的维度，这个网络以原始视频为输入，并输出一个在时间和空间上都被压缩的潜状态表示。Sora在这个压缩的潜空间内进行训练，然后生成视频。此外还训练了一个对应的解码器模型，它可以将生成的潜态映射回像素空间。这一部分类似LDM中VAE编解码模块，但不同的是可以直接对整个视频压缩，并同时适用图像模态，与之接近的技术是W.A.L.T和MAGVIT-v2。
<div align="center">
<img  alt="image" src="https://private-user-images.githubusercontent.com/20476674/308176534-bf5231e1-754a-41f5-93c5-9ae1583d20d7.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjEwNDI1MTUsIm5iZiI6MTcyMTA0MjIxNSwicGF0aCI6Ii8yMDQ3NjY3NC8zMDgxNzY1MzQtYmY1MjMxZTEtNzU0YS00MWY1LTkzYzUtOWFlMTU4M2QyMGQ3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA3MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNzE1VDExMTY1NVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWEzMjUwMGI4YzhlNzg0NDY4ZGMyYThjYTE2MThmZGZkMTEzMmI4M2E2ZjJhMzFkNGZjMmE1Y2EwNTJiYWM5MDEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.rqFnyfLEG0sZesjmsvEZXJd7_sQtE_WZb_oB1KWxlSo">
</div>

PPDiffuser已支持MAGVIT-v2，它是一个视频编码器，通过Cascual 3D CNN和Lookup-Free Quantization (LFQ)来为视频和图像进行联合编码。具体请参考[ppdiffusers/examples/video_tokenizer/magvit2](https://github.com/PaddlePaddle/PaddleMIX/tree/upgrade_ppdiffusers0240/ppdiffusers/examples/video_tokenizer/magvit2)。




## patchify技术（patches and variable durations, resolutions, aspect ratios）
给定一个视频压缩网络压缩后的输入视频，Sora会提取一系列的spacetime patches（时空区块）作为transformer tokens。基于patches的表示法使得Sora能够在不同分辨率、持续时间和宽高比的视频和图像上进行训练。在推理时，可以通过在指定大小的grid中随机初始化这些patches来控制生成视频的大小。patches是Sora的核心改进，其灵感来自于大型语言模型。大型语言模型（LLM）成功的部分原因在于，它们使用了分词（tokens）来优雅地统一文本的各种形式——代码、数学和各种自然语言。就像LLM有文本tokens一样，Sora有视觉patches，对于训练处理各种类型的视频和图像的生成模型来说，patches是一种高度可扩展且有效的表示方式。不同于过去的图像和视频生成方法通常会将视频调整大小、裁剪或修剪到标准尺寸，例如，256x256分辨率下的4秒视频，通过patchify相关处理，Sora可以在原始尺寸的数据上进行大规模训练。与之相关的技术是NaViT(Patch n'Pack)。
<div align="center">
<img  alt="image" src="https://private-user-images.githubusercontent.com/20476674/308177896-60259426-2252-4447-8e03-b91022d883cb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjEwNDI1MTUsIm5iZiI6MTcyMTA0MjIxNSwicGF0aCI6Ii8yMDQ3NjY3NC8zMDgxNzc4OTYtNjAyNTk0MjYtMjI1Mi00NDQ3LThlMDMtYjkxMDIyZDg4M2NiLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA3MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNzE1VDExMTY1NVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTNmNmMzYjY0NDM5MGQyMDcwMDkxYzA0NzA3MGMwYjhhOTAyOThiMTc3YWU4ZWJjNTg0ODJmYzVkMTRiOTkxZWUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.vX-Rw-YLz95Z1kWMb74qUhL7BqePcMD_g4YeDtPwTsU">
</div>

PPDiffuser已支持NaViT，它将图像视为patches并从不同图像中获取的多个paches打包在一个序列中，通过这种Patch n’ Pack的方式使得图像或视频的预训练可以适应不同的时长、长宽比、分辨率，提高训推性能和灵活性。具体请参考[ppdiffusers/examples/navit](https://github.com/PaddlePaddle/PaddleMIX/tree/upgrade_ppdiffusers0240/ppdiffusers/examples/navit)。


## diffusion transformer
Sora是一个扩散模型，在给定噪声patches（以及像文本提示这样的条件信息）的输入下，它能够被训练去预测原始的图像块。与传统UNet based的方式不同的是，Sora是一个Transformer-based的扩散模型，也即diffusion transformer（DiT）。transformer在多个领域展示出了显著的扩展性能，包括语言建模，计算机视觉和图像生成。SORA发现diffusion transformer也能对视频模型进行有效地扩展。在训练过程中，固定种子和输入的视频样本，随着训练计算量的增加，样本质量也显著提高。与之相关的技术是transformer-based扩散模型，如DiT、SiT、UViT等。

<div align="center">
<img  alt="image" src="https://private-user-images.githubusercontent.com/20476674/308176745-3cc90a02-d498-4ab5-a671-b475a02bf37c.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjEwNDI1MTUsIm5iZiI6MTcyMTA0MjIxNSwicGF0aCI6Ii8yMDQ3NjY3NC8zMDgxNzY3NDUtM2NjOTBhMDItZDQ5OC00YWI1LWE2NzEtYjQ3NWEwMmJmMzdjLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA3MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNzE1VDExMTY1NVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTk3ZTQxOWFjOTk4YjU3MzIxNWQ5ZmQ3ODFiOTdjZGZlZTY4NmFmMDRmNjhkZjFkYmZkNzg1MzZmMjU1NzJhNzAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.2mGQfePqxoLQJJonnjpFEhCnsm4_hwD094CpAnIKHlg">
</div>

PPDiffuser已支持DiT、SiT、UViT。在DiT中，首先将输入潜变量分解成若干个patches，然后由多个DiT block进行处理，并通过去噪过程完成图片的生成，SiT在模型结构上同DiT一致，针对扩散过程进行改进，这两个模型具体请参考[ppdiffusers/examples/class_conditional_image_generation/DiT](https://github.com/PaddlePaddle/PaddleMIX/tree/upgrade_ppdiffusers0240/ppdiffusers/examples/class_conditional_image_generation/DiT)。
UViT是DiT同期工作，它引入跳连结构，基于此构建的UniDiffuser通过统一的多模态扩散过程支持文生图、图生文等任务，PPDiffusers已全面支持[ppdiffusers/examples/uvit](https://github.com/PaddlePaddle/PaddleMIX/tree/upgrade_ppdiffusers0240/ppdiffusers/examples/text_to_image_uvit_coco)。



## 语言理解（language understanding）
训练文本到视频生成系统需要大量带有相应文本标题的视频。OpenAI将在DALL·E 3中引入的再标记技术应用于视频。首先训练一个描述性极强的captioner model，然后使用它为训练集中的所有视频生成文本标题。结果发现，在高度描述性的视频标题上进行训练可以提高文本的准确度以及视频的整体质量。在推理时，与DALL·E 3类似，也利用GPT将用户的短提示转化为更长、更详细的标题。这使得Sora能够生成准确遵循用户提示的高质量视频。与之相关的技术是预训练图文大模型。

PaddleMIX已支持多款预训练图文大模型，具体请参考[paddlemix/examples](https://github.com/PaddlePaddle/PaddleMIX/tree/upgrade_ppdiffusers0240/paddlemix/examples)。

## 参考
- https://openai.com/sora
- https://openai.com/research/video-generation-models-as-world-simulators