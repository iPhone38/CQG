# 易错判断题生成与预测系统

一个基于DeepSeek API的判断题生成与多模型预测系统，能够生成易错判断题并预测不同年级学生的答题正确率。

## 功能展示
![生成界面](https://github.com/iPhone38/CQG/blob/main/asset/generation.png)

![预测界面](https://github.com/iPhone38/CQG/blob/main/asset/prediction.png)

## 功能特点

* 🎯 生成具有迷惑性的易错判断题
* 📊 预测幼儿园到研究生各年级学生的答题正确率
* 📈 生成详细的统计报告和可视化结果
* 🎨 直观的图形用户界面
* ⚡ 多线程处理，避免界面卡顿
* 🔒 支持环境变量配置API密钥

## 环境要求

* Python 3.7+
* DeepSeek API密钥

## 安装步骤

1. 克隆或下载项目文件
2. 安装依赖包：

**bash**

```
pip install -r requirements.txt
```

3. 获取DeepSeek API密钥：
   * 访问 [DeepSeek官网](https://platform.deepseek.com/)
   * 注册账号并获取API密钥

## API密钥配置方式

### 方式一：在GUI界面中直接输入（推荐用于本地测试）

运行程序后，在界面中直接输入您的DeepSeek API密钥。

### 方式二：使用环境变量（推荐用于生产环境）

#### 本地开发环境配置

在项目根目录创建 `.env` 文件：

```bash
DEEPSEEK_API_KEY=your_api_key_here
```

**注意**：`.env` 文件包含敏感信息，请勿提交到版本控制系统。建议在 `.gitignore` 中添加：
```
.env
```

#### GitHub Actions 配置（使用 Repository Secrets）

如果您需要在GitHub Actions中运行此程序，可以使用GitHub Repository Secrets来安全地存储API密钥：

**步骤 1：找到 Repository Secrets 设置**

1. 进入您的GitHub仓库页面
2. 点击仓库顶部的 **Settings**（设置）选项卡
3. 在左侧导航栏中找到 **Security**（安全）部分
4. 点击 **Secrets and variables** → **Actions**
5. 点击 **New repository secret** 按钮（这就是"New repository secret按钮"的位置！）

**步骤 2：添加 Secret**

1. **Name（名称）**：输入 `DEEPSEEK_API_KEY`
2. **Secret（密钥值）**：粘贴您的DeepSeek API密钥
3. 点击 **Add secret**（添加密钥）按钮保存

**步骤 3：在GitHub Actions中使用**

在您的workflow文件（`.github/workflows/*.yml`）中引用：

```yaml
- name: Run application
  env:
    DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
  run: python judgment_question_gui.py
```

**可视化说明：**

```
GitHub仓库页面
  ↓
Settings（设置）
  ↓
左侧导航：Security → Secrets and variables → Actions
  ↓
点击"New repository secret"按钮 ← 这里！
  ↓
填写Name和Secret
  ↓
点击"Add secret"完成
```

## 使用方法

1. 运行程序：

**bash**

```
python judgment_question_gui.py
```

2. 在界面中输入：
   * 您的DeepSeek API密钥（如果已配置环境变量，会自动填充）
   * 想要生成判断题的主题描述（如"小学数学几何知识"）
   * 题目数量（1-20道）
3. 点击"生成题目"按钮开始生成判断题
4. 生成完成后，点击"开始多年级预测"按钮进行多模型预测
5. 查看结果：
   * **原始输出** ：API返回的原始题目格式
   * **格式化输出** ：整理后的题目和预测结果
   * **统计报告** ：详细的统计分析
   * **结果表格** ：各年级预测正确率的可视化表格

## 项目结构

**text**

```
judgment-question-system/
├── judgment_question_gui.py  # 主程序文件
├── requirements.txt          # 依赖包列表
├── .env.example             # 环境变量示例文件（可选）
└── README.md                # 说明文档
```

## 安全注意事项

* ⚠️ **切勿将API密钥提交到版本控制系统**
* 🔒 使用 `.env` 文件存储本地API密钥，并将其添加到 `.gitignore`
* 🔐 在GitHub等平台上使用Repository Secrets功能存储敏感信息
* 🛡️ 定期轮换API密钥以提高安全性
* 👥 不要在公开场合（如截图、日志）中暴露API密钥

## 注意事项

* 确保网络连接正常，能够访问DeepSeek API
* API调用需要消耗相应的token配额
* 生成大量题目或多年级预测可能需要较长时间
* 建议先从少量题目开始测试

## 技术支持

如遇到问题，请检查：

1. API密钥是否正确
2. 网络连接是否正常
3. 依赖包是否安装完整
4. 环境变量是否正确配置（如果使用环境变量方式）

## 常见问题

### Q: "New repository secret按钮在哪？"
**A:** 在GitHub仓库中：Settings → Security → Secrets and variables → Actions → New repository secret

### Q: 如何确认API密钥已经加载？
**A:** 如果配置了环境变量 `DEEPSEEK_API_KEY`，程序启动时API密钥输入框会自动填充（显示为密码形式）

### Q: .env文件应该放在哪里？
**A:** 放在项目根目录（与 `judgment_question_gui.py` 同级目录）

## 许可证

MIT License

## 更新日志

### v1.1.0
* 新增环境变量支持，可通过 `DEEPSEEK_API_KEY` 配置API密钥
* 添加GitHub Repository Secrets配置说明
* 增强文档，详细说明API密钥的多种配置方式
* 改进安全性，提醒用户不要提交敏感信息

### v1.0.0

* 初始版本发布
* 实现判断题生成功能
* 实现多年级预测功能
* 添加图形用户界面
* 生成统计报告和可视化表格
