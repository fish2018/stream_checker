# 直播源链接检测工具

这是一个批量异步并发检测直播源链接有效性的工具。它可以从多种格式的文件中提取URL，并使用异步并发技术快速检测这些链接的有效性。

## 功能特点

- 支持多种文件格式（TXT、M3U/M3U8）
- 自动提取和检测直播源URL
- 异步并发请求，高效处理大量URL
- 智能识别直播源名称和分类
- 详细的结果报告，包括有效性和响应时间
- 支持递归扫描子目录
- 可配置的并发数和超时时间
- 结果分类保存，方便使用
- **批量保存结果**：每处理100个URL后自动保存结果，防止程序中断导致数据丢失
- **断点续传功能**：记录处理进度，下次运行时可以从上次中断的地方继续
- **智能过滤**：自动过滤掉包含"jar"的非直播源链接
- **自动编号**：对重复的直播台名称自动添加编号，确保播放器能识别所有链接
- **速度优化**：根据响应速度排序，将快的线路放在前面
- **智能聚合**：将类似名称的电台聚合在一起，方便查找
- **链接清理**：自动处理带有#符号的链接，只保留#符号前面的有效部分
- **繁简转换**：自动将繁体中文转换为简体中文，方便大陆用户使用
- **全局排序**：对最终结果文件中的链接按类别和名称进行全局排序
- **自动清理**：结束后自动删除所有临时文件，只保留一个最终整理好的文件
- **深度验证**：对m3u8格式的直播流进行深度验证，确保链接真正可播放
- **多格式验证**：支持验证多种流媒体格式，包括m3u8、flv、ts、mp4等

## 安装

1. 确保已安装Python 3.7或更高版本
2. 安装依赖包：

```bash
pip install aiohttp tqdm opencc-python-reimplemented urllib3
```

## 使用方法

### 修改脚本变量

打开 `stream_checker.py` 文件，在文件顶部找到配置部分，直接修改变量值：

```python
# =====================================================================
# 配置部分 - 直接修改这里的变量来配置程序
# =====================================================================

# 要扫描的目录路径，使用绝对路径或相对路径
DIRECTORY = "."

# 要包含的文件扩展名列表
EXTENSIONS = [".txt", ".m3u", ".m3u8"]

# 最大并发请求数
CONCURRENCY = 200

# 请求超时时间（秒）
TIMEOUT = 10

# 是否递归扫描子目录
RECURSIVE = True

# 结果保存文件路径
OUTPUT_FILE = f"results_{time.strftime('%Y%m%d_%H%M%S')}.json"

# 自定义 User-Agent
USER_AGENT = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"

# 断点续传文件路径
CHECKPOINT_FILE = "stream_checker_checkpoint.pkl"

# 是否启用断点续传
ENABLE_CHECKPOINT = True

# 每处理多少个URL保存一次结果
SAVE_INTERVAL = 100

# 是否将繁体字转换为简体字
CONVERT_TO_SIMPLIFIED = True

# 是否只保留一个最终输出文件
KEEP_ONLY_FINAL_FILE = True

# 是否在最终文件中包含聚合分组部分
INCLUDE_AGGREGATED_SECTION = False

# 是否对m3u8文件进行深度验证
DEEP_VALIDATION = True

# 验证m3u8文件时检查的分片数量
M3U8_SEGMENTS_TO_CHECK = 3

# 是否验证m3u8文件的内容格式
VALIDATE_M3U8_CONTENT = True

# 是否验证其他格式的流媒体
VALIDATE_OTHER_STREAMS = True

# 是否使用ffprobe进行高级验证（需要安装ffmpeg）
USE_FFPROBE = False
```

> **注意**：虽然默认配置中的 `EXTENSIONS` 不包含 `.json`，但程序内部在搜索文件时会自动包含 `.json` 文件，并且能够处理JSON格式的内容。如果您想明确包含JSON文件，可以将配置修改为：`EXTENSIONS = [".txt", ".m3u", ".m3u8", ".json"]`

修改完成后，直接运行脚本：

```bash
python stream_checker.py
```

## 配置说明

- `DIRECTORY`: 要扫描的目录路径，可以是相对路径或绝对路径
- `EXTENSIONS`: 要包含的文件扩展名列表，默认为 [".txt", ".m3u", ".m3u8"]。注意：即使不在此列表中指定，程序也会自动处理 `.json` 文件
- `CONCURRENCY`: 最大并发请求数，默认为 200
- `TIMEOUT`: 请求超时时间（秒），默认为 10
- `RECURSIVE`: 是否递归扫描子目录，默认为 True
- `OUTPUT_FILE`: 结果保存文件路径，默认为 "results_当前时间.json"
- `USER_AGENT`: 自定义 User-Agent 字符串
- `CHECKPOINT_FILE`: 断点续传文件路径
- `ENABLE_CHECKPOINT`: 是否启用断点续传
- `SAVE_INTERVAL`: 每处理多少个URL保存一次中间结果和检查点
- `CONVERT_TO_SIMPLIFIED`: 是否将繁体字转换为简体字
- `KEEP_ONLY_FINAL_FILE`: 是否只保留最终排序后的文件，删除中间文件
- `INCLUDE_AGGREGATED_SECTION`: 是否在最终文件中包含聚合分组部分，默认为否，可以减小文件大小
- `DEEP_VALIDATION`: 是否对m3u8文件进行深度验证，默认为是，可以更准确地检测直播流是否可播放
- `M3U8_SEGMENTS_TO_CHECK`: 验证m3u8文件时检查的分片数量，默认为3
- `VALIDATE_M3U8_CONTENT`: 是否验证m3u8文件的内容格式，默认为是
- `VALIDATE_OTHER_STREAMS`: 是否验证其他格式的流媒体（如FLV、PHP动态生成的流等），默认为是
- `USE_FFPROBE`: 是否使用ffprobe进行高级验证，默认为否（需要安装ffmpeg）

## 输出文件

程序会生成以下文件：

1. JSON结果文件（默认：results_YYYYMMDD_HHMMSS.json）：包含所有检测结果和统计信息
2. 有效URL文本文件（默认：results_YYYYMMDD_HHMMSS_valid.txt）：包含有效的URL和直播名称，按类别分组
3. 聚合分组文件（默认：results_YYYYMMDD_HHMMSS_grouped.txt）：将类似名称的电台聚合在一起，并按响应速度排序
4. 最终排序文件（默认：results_YYYYMMDD_HHMMSS_final.txt）：包含所有有效链接，按类别和名称全局排序

如果启用了`KEEP_ONLY_FINAL_FILE`选项（默认启用），程序结束后会自动删除前三个中间文件，只保留最终排序文件，使结果更加整洁。

### 最终排序文件格式

最终排序文件包含以下内容：

1. 按类别排序的部分：所有链接按类别名称字母顺序排序，同一类别内的链接按名称字母顺序排序
2. 聚合分组部分（可选）：将类似名称的电台聚合在一起，方便查找（仅当`INCLUDE_AGGREGATED_SECTION=True`时包含）

```
类别A,#genre#
频道A,http://example.com/streamA
频道B 1,http://example.com/streamB1
频道B 2,http://example.com/streamB2

类别B,#genre#
频道C,http://example.com/streamC
频道D,http://example.com/streamD

# 以下部分仅当INCLUDE_AGGREGATED_SECTION=True时包含
聚合分组,#genre#
频道A,http://example.com/streamA
频道B 1,http://example.com/streamB1
频道B 2,http://example.com/streamB2
频道C,http://example.com/streamC
频道D,http://example.com/streamD
```

这种格式既提供了按类别组织的视图，也可以选择性地包含按名称相似度聚合的视图。默认情况下，为了减小文件大小，不包含聚合分组部分。

### 有效URL文本文件格式

有效URL文本文件采用以下格式：

```
类别1,#genre#
直播名称1,http://example.com/stream1
直播名称2 1,http://example.com/stream2a
直播名称2 2,http://example.com/stream2b

类别2,#genre#
直播名称3,http://example.com/stream3
直播名称4,http://example.com/stream4
```

注意：对于同名的直播源，会自动添加编号（如"直播名称2 1"和"直播名称2 2"），确保播放器能识别所有链接。

### 聚合分组文件格式

聚合分组文件将类似名称的电台聚合在一起，并按响应速度排序：

```
聚合分组,#genre#
CCTV1,http://example.com/cctv1-fast
CCTV1 2,http://example.com/cctv1-slow
CCTV2,http://example.com/cctv2
...

央视,#genre#
CCTV1,http://example.com/cctv1-fast
CCTV1 2,http://example.com/cctv1-slow
CCTV2,http://example.com/cctv2
...
```

这种格式既提供了按名称相似度聚合的视图，也保留了原始的分类组织。

## 智能过滤和排序

程序包含以下智能处理功能：

1. **自动过滤非直播源**：
   - 忽略包含"jar"的链接，这些通常不是直播源
   - 过滤掉明显无效的URL格式

2. **智能排序**：
   - 同一直播台的多个链接按响应速度排序，将速度快的链接放在前面
   - 这确保用户优先使用最流畅的线路

3. **智能聚合**：
   - 将名称相似的直播台聚合在一起（基于名称前缀）
   - 生成额外的聚合分组文件，方便查找相关频道

4. **自动编号**：
   - 对同名直播台自动添加编号（如"CCTV1 1"、"CCTV1 2"）
   - 确保播放器能正确识别和显示所有链接

5. **链接清理**：
   - 自动处理带有#符号的链接，只保留#符号前面的有效部分
   - 例如："http://example.com/stream#http://backup.com/stream" 会被处理为 "http://example.com/stream"
   - 这避免了因注释或备用链接导致的播放问题

6. **繁简转换**：
   - 自动将频道名称和分类中的繁体中文转换为简体中文
   - 使直播源列表更适合大陆用户使用
   - 需要安装 opencc-python-reimplemented 库才能使用此功能

7. **全局排序**：
   - 对最终结果文件中的链接按类别和名称进行全局排序
   - 类别按字母顺序排序，同一类别内的频道按名称字母顺序排序
   - 使结果文件更加有序，方便查找特定频道

8. **自动清理**：
   - 程序结束后自动删除所有临时文件，只保留一个最终整理好的文件
   - 避免生成过多中间文件占用磁盘空间
   - 可通过`KEEP_ONLY_FINAL_FILE`选项控制是否启用此功能

9. **深度验证**：
   - 对m3u8格式的直播流进行深度验证，确保链接真正可播放
   - 检查m3u8文件的内容格式是否正确
   - 验证主播放列表中的变体流是否可访问
   - 检查媒体播放列表中的分片是否可访问
   - 这大大提高了检测结果的准确性，确保输出的链接都能正常播放

10. **多格式验证**：
    - 支持验证多种流媒体格式，包括m3u8、flv、ts、mp4等
    - 对FLV格式进行专门的文件头验证，确保格式正确
    - 识别PHP动态生成的流媒体内容
    - 通过URL模式匹配识别常见的流媒体链接格式
    - 可选择使用ffprobe进行更高级的流媒体验证（需要安装ffmpeg）

这些功能使得生成的直播源列表更加整洁、实用，特别适合导入到各种IPTV播放器中使用。

## 输出示例

程序运行完成后，会显示如下摘要信息：

```
Summary:
Total URLs: 1000
Valid: 750 (75.0%)
Invalid: 150 (15.0%)
Timeout: 50 (5.0%)
Error: 50 (5.0%)
Total time: 120.45 seconds
Detailed results saved to results_20230415_123456.json
Valid URLs with names saved to results_20230415_123456_valid.txt
```

## 支持的文件格式

### TXT 文件格式

支持以下格式的TXT文件：

1. 名称,URL 格式：
```
CCTV1,http://example.com/cctv1
CCTV2,http://example.com/cctv2
```

2. 带类别的格式：
```
央视,#genre#
CCTV1,http://example.com/cctv1
CCTV2,http://example.com/cctv2

卫视,#genre#
湖南卫视,http://example.com/hunan
江苏卫视,http://example.com/jiangsu
```

### M3U/M3U8 文件格式

支持标准的M3U/M3U8格式：

```
#EXTM3U
#EXTINF:-1 tvg-name="CCTV1",CCTV1
http://example.com/cctv1
#EXTINF:-1 tvg-name="CCTV2",CCTV2
http://example.com/cctv2
```

### JSON 文件格式

虽然默认配置中的 `EXTENSIONS` 不包含 `.json`，但程序内部会自动处理JSON文件。支持各种包含URL的JSON格式，会尝试从JSON中提取名称和URL对。例如：

```json
{
  "channels": [
    {
      "name": "CCTV1",
      "url": "http://example.com/cctv1"
    },
    {
      "name": "CCTV2",
      "url": "http://example.com/cctv2"
    }
  ]
}
```

## 断点续传功能

如果程序在处理过程中被中断（如按Ctrl+C或系统关机），下次运行时会自动从上次中断的地方继续处理。这是通过以下机制实现的：

1. 程序会在处理过程中定期保存检查点文件（默认为`stream_checker_checkpoint.pkl`）
2. 检查点文件记录了已处理的URL和当前结果
3. 下次运行时，程序会检查是否存在检查点文件，如果存在则加载并跳过已处理的URL
4. 处理完成后，检查点文件会被自动删除

您可以通过以下配置选项控制断点续传功能：

- `ENABLE_CHECKPOINT`：设置为`True`启用断点续传，设置为`False`禁用
- `CHECKPOINT_FILE`：检查点文件的路径
- `SAVE_INTERVAL`：每处理多少个URL保存一次中间结果和检查点

## 批量保存结果

为防止长时间运行过程中意外中断导致数据丢失，程序会定期保存中间结果：

1. 每处理指定数量的URL（默认为100个）后，会自动保存当前结果到输出文件
2. 您可以通过修改`SAVE_INTERVAL`配置选项来调整保存频率

## 注意事项

- 处理大量URL可能需要较长时间，请耐心等待
- 如果遇到网络问题，可以尝试增加超时时间或减少并发数
- 对于特定网站，可能需要自定义User-Agent
- 程序会自动跳过重复的URL，以提高效率
- 深度验证功能可能会增加处理时间，但能显著提高检测准确性
- 如果需要验证RTMP流，建议启用ffprobe高级验证（需要安装ffmpeg）
