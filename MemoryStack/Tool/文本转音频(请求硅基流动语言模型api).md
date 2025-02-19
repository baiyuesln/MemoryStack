#tts

	因为有一些需要反复理解的文字内容我想转为音频,以便在做一些家务的时候加深理解

```py
import os
import requests
from tqdm import tqdm  # 进度条支持（可选）

# ========= 配置区 =========
INPUT_DIR = "/Users/majic_li/Documents/input"    # 存放 txt 文件的文件夹
OUTPUT_DIR = "/Users/majic_li/Documents/output/"  # 音频输出目录
API_TOKEN = "sk-jryyglokhbckcctjspafsxuldluxjjcldnztopsyjlvvgxid"  # 替换成你的真实 token
# ==========================

# 创建输出目录（如果不存在）
os.makedirs(OUTPUT_DIR, exist_ok=True)
os.makedirs(INPUT_DIR, mode=0o755, exist_ok=True)
if not os.path.exists(INPUT_DIR):
    print(f"❌ 输入目录不存在: {INPUT_DIR}")
    exit()

if not os.access(INPUT_DIR, os.R_OK):
    print(f"❌ 无权访问目录: {INPUT_DIR}")
    exit()

# 获取所有 txt 文件列表（自动跳过子目录）
txt_files = [
    f for f in os.listdir(INPUT_DIR) 
    if f.endswith(".txt") and os.path.isfile(os.path.join(INPUT_DIR, f))
]

if not txt_files:
    print(f"❌ 输入目录 {INPUT_DIR} 中找不到任何 txt 文件！")
    exit()

# 带进度条循环（安装 tqdm：pip install tqdm）
for txt_file in tqdm(txt_files, desc="转换进度"):
    # 构建完整文件路径
    input_path = os.path.join(INPUT_DIR, txt_file)
    output_path = os.path.join(OUTPUT_DIR, os.path.splitext(txt_file)[0] + ".mp3")

    try:
        # 步骤1: 读取 txt 内容
        with open(input_path, "r", encoding="utf-8") as f:
            text_content = f.read().strip()
            if not text_content:
                print(f"\n⚠️ 文件 {txt_file} 内容为空，跳过处理")
                continue

        # 步骤2: 发送API请求
        response = requests.post(
            url="https://api.siliconflow.cn/v1/audio/speech",
            headers={
                "Authorization": f"Bearer {API_TOKEN}",
                "Content-Type": "application/json"
            },
            json={
                "model": "fishaudio/fish-speech-1.5",
                "input": text_content,
                "voice": "fishaudio/fish-speech-1.5:bella",
                "response_format": "mp3",
                "stream": True,
                "sample_rate": 32000
            },
            stream=True
        )
        
        # 步骤3: 处理响应
        if response.status_code != 200:
            print(f"\n🚫 转换失败：{txt_file} | 状态码：{response.status_code} | 错误信息：{response.text[:200]}")
            continue

        # 步骤4: 保存音频
        with open(output_path, "wb") as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)

    except Exception as e:
        print(f"\n🔥 处理文件 {txt_file} 时出现异常：{str(e)}")

print(f"\n✅ 转换完成！共处理 {len(txt_files)} 个文件，输出到 {OUTPUT_DIR} 目录")
```