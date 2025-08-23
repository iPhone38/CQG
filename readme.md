# 易错判断题生成与预测系统

一个基于DeepSeek API的判断题生成与多模型预测系统，能够生成易错判断题并预测不同年级学生的答题正确率。

## 功能展示

```
![生成界面](https://example.com/path/to/generation.png)
```

```
![预测界面](https://example.com/path/to/prediction.png)
```

## 功能特点

* 🎯 生成具有迷惑性的易错判断题
* 📊 预测幼儿园到研究生各年级学生的答题正确率
* 📈 生成详细的统计报告和可视化结果
* 🎨 直观的图形用户界面
* ⚡ 多线程处理，避免界面卡顿

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

## 使用方法

1. 运行程序：

**bash**

```
python judgment_question_gui.py
```

2. 在界面中输入：
   * 您的DeepSeek API密钥
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
└── README.md                # 说明文档
```

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

## 许可证

MIT License

## 更新日志

### v1.0.0

* 初始版本发布
* 实现判断题生成功能
* 实现多年级预测功能
* 添加图形用户界面
* 生成统计报告和可视化表格
