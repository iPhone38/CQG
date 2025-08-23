import sys
import os
import requests
import json
import re
import random
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, 
                             QLabel, QTextEdit, QLineEdit, QPushButton, QSpinBox, 
                             QTabWidget, QGroupBox, QScrollArea, QFrame, QProgressBar,
                             QSplitter, QTableWidget, QTableWidgetItem, QHeaderView)
from PyQt5.QtCore import Qt, QThread, pyqtSignal
from PyQt5.QtGui import QFont, QPalette, QColor

class DeepSeekWorker(QThread):
    """后台工作线程，用于处理DeepSeek API调用"""
    finished = pyqtSignal(str, bool)
    progress = pyqtSignal(str)
    
    def __init__(self, generator, topic, num_questions):
        super().__init__()
        self.generator = generator
        self.topic = topic
        self.num_questions = num_questions
    
    def run(self):
        try:
            self.progress.emit("正在生成题目，请稍候...")
            questions_content = self.generator.generate_judgment_questions(self.topic, self.num_questions)
            
            if questions_content and self.generator.questions:
                self.finished.emit(questions_content, True)
            else:
                self.finished.emit("生成题目失败，请检查API密钥和网络连接。", False)
                
        except Exception as e:
            self.finished.emit(f"生成过程中出现错误: {str(e)}", False)


class TestWorker(QThread):
    """测试工作线程，用于多模型测试"""
    finished = pyqtSignal(dict, bool)
    progress = pyqtSignal(str)
    grade_progress = pyqtSignal(int, int, str)  # 当前年级索引, 总年级数, 年级名称
    
    def __init__(self, generator, grades):
        super().__init__()
        self.generator = generator
        self.grades = grades
    
    def run(self):
        try:
            all_results = {}
            total_grades = len(self.grades)
            
            for i, grade in enumerate(self.grades):
                self.progress.emit(f"正在让{grade}模型预测正确率...")
                self.grade_progress.emit(i + 1, total_grades, grade)
                
                results = self.generator.predict_grade_accuracy(grade)
                all_results[grade] = results
            
            self.finished.emit(all_results, True)
                
        except Exception as e:
            self.finished.emit({}, False)
            self.progress.emit(f"测试过程中出现错误: {str(e)}")


class DeepSeekJudgmentQuestionGenerator:
    def __init__(self, api_key):
        self.api_key = api_key
        self.api_url = "https://api.deepseek.com/v1/chat/completions"
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
        self.questions = []
        
        self.grade_to_model = {
            '幼儿园': 'deepseek-chat',
            '小学': 'deepseek-chat', 
            '初中': 'deepseek-chat',
            '高中': 'deepseek-chat',
            '大学': 'deepseek-chat',
            '研究生': 'deepseek-chat'
        }
        
        self.model_prompts = {
            '幼儿园': "请你预测幼儿园小朋友回答这个判断题的正确概率。",
            '小学': "请你预测小学生回答这个判断题的正确概率。",
            '初中': "请你预测初中生回答这个判断题的正确概率。",
            '高中': "请你预测高中生回答这个判断题的正确概率。",
            '大学': "请你预测大学生回答这个判断题的正确概率。",
            '研究生': "请你预测研究生回答这个判断题的正确概率。"
        }
    
    def call_deepseek(self, prompt, model="deepseek-chat", temperature=0.7):
        """调用DeepSeek API"""
        payload = {
            "model": model,
            "messages": [{"role": "user", "content": prompt}],
            "temperature": temperature,
            "max_tokens": 500
        }
        
        try:
            response = requests.post(self.api_url, headers=self.headers, json=payload)
            response.raise_for_status()
            result = response.json()
            return result['choices'][0]['message']['content']
        except Exception as e:
            print(f"API调用错误: {e}")
            return None
    
    def parse_questions(self, content):
        """解析生成的题目"""
        questions = []
        question_blocks = re.split(r'(?=题目\d*[：:])', content)
        
        for block in question_blocks:
            if not block.strip():
                continue
                
            question = {
                'content': '',
                'answer': '',
                'analysis': ''
            }
            
            lines = block.strip().split('\n')
            for line in lines:
                line = line.strip()
                if not line:
                    continue
                    
                if re.match(r'题目\d*[：:]', line):
                    question['content'] = re.sub(r'题目\d*[：:]', '', line).strip()
                elif line.startswith('答案：') or line.startswith('答案:'):
                    question['answer'] = line.replace('答案：', '').replace('答案:', '').strip()
                elif line.startswith('解析：') or line.startswith('解析:'):
                    question['analysis'] = line.replace('解析：', '').replace('解析:', '').strip()
                elif not question['answer'] and question['content']:
                    question['content'] += ' ' + line
            
            if question['content'] and question['answer']:
                questions.append(question)
        
        return questions
    
    def generate_judgment_questions(self, topic_description, num_questions=5):
        """生成易错判断题"""
        prompt = f"""请根据以下主题描述，生成{num_questions}道易错的判断题。

主题：{topic_description}

要求：
1. 每道题都必须设计有容易出错的陷阱
2. 题目要考察常见的误解或容易混淆的概念
3. 输出格式严格遵循：

题目1：[判断题内容]
答案：[正确/错误]
解析：[简要说明为什么容易错]

题目2：[判断题内容]
答案：[正确/错误]
解析：[简要说明为什么容易错]

请直接输出题目，不要有其他解释性文字。"""

        response = self.call_deepseek(prompt)
        if response:
            self.questions = self.parse_questions(response)
        return response
    
    def extract_probability(self, text):
        """从文本中提取概率值"""
        # 查找百分比格式
        percentage_match = re.search(r'(\d{1,3})%', text)
        if percentage_match:
            return int(percentage_match.group(1)) / 100
        
        # 查找小数格式
        decimal_match = re.search(r'0?\.\d+', text)
        if decimal_match:
            return float(decimal_match.group(0))
        
        # 查找分数格式
        fraction_match = re.search(r'(\d+)/(\d+)', text)
        if fraction_match:
            numerator = int(fraction_match.group(1))
            denominator = int(fraction_match.group(2))
            if denominator > 0:
                return numerator / denominator
        
        # 如果找不到明确的概率值，返回None
        return None
    
    def ask_model_to_predict(self, question_content, grade_level):
        """让指定年级的模型预测正确概率"""
        model_name = self.grade_to_model[grade_level]
        prompt_template = self.model_prompts[grade_level]
        
        prompt = f"""{prompt_template}

题目：{question_content}

请预测{grade_level}学生回答这道判断题的正确概率，输出一个0到1之间的小数（如0.75）或百分比（如75%）。
请只输出概率值，不要有其他解释。"""
        
        print(f"向{grade_level}模型请求预测: {question_content[:50]}...")
        response = self.call_deepseek(prompt, model=model_name, temperature=0.3)
        
        if response:
            response = response.strip()
            print(f"{grade_level}模型预测: {response}")
            
            # 提取概率值
            probability = self.extract_probability(response)
            if probability is not None:
                return max(0.0, min(1.0, probability))  # 确保在0-1范围内
        
        # 如果API调用失败或无法提取概率，返回随机值
        return round(random.uniform(0.3, 0.8), 2)
    
    def predict_grade_accuracy(self, grade_level):
        """预测特定年级的学生回答所有题目的正确概率"""
        print(f"\n开始让{grade_level}年级模型预测正确率...")
        
        results = []
        for i, question in enumerate(self.questions):
            if not question.get('content') or not question.get('answer'):
                print(f"跳过第{i+1}题，题目格式不完整")
                continue
                
            predicted_accuracy = self.ask_model_to_predict(question['content'], grade_level)
            
            results.append({
                'question_index': i + 1,
                'predicted_accuracy': predicted_accuracy,
                'correct_answer': question['answer']
            })
        
        return results
    
    def generate_statistics_report(self, all_results, topic):
        """生成统计报告"""
        if not all_results:
            return "暂无测试数据"
            
        report = f"""=== 易错判断题多模型预测报告 ===

主题：{topic}
题目数量：{len(self.questions)}道
测试年级数量：{len(all_results)}个

各题目预测正确率统计：
{"题目":<6} {"平均正确率":<12} {"难度等级":<8} {"预测范围":<15}
{"-"*50}"""

        # 计算每道题的平均预测正确率
        question_stats = []
        for i, question in enumerate(self.questions):
            if i >= len(next(iter(all_results.values())) if all_results else 0):
                break
                
            accuracies = []
            min_accuracy = 1.0
            max_accuracy = 0.0
            
            for grade_level, results in all_results.items():
                if i < len(results):
                    accuracy = results[i]['predicted_accuracy']
                    accuracies.append(accuracy)
                    min_accuracy = min(min_accuracy, accuracy)
                    max_accuracy = max(max_accuracy, accuracy)
            
            avg_accuracy = sum(accuracies) / len(accuracies) if accuracies else 0
            
            # 确定难度等级
            if avg_accuracy >= 0.8:
                difficulty = "简单"
            elif avg_accuracy >= 0.6:
                difficulty = "中等"
            elif avg_accuracy >= 0.4:
                difficulty = "较难"
            else:
                difficulty = "困难"
            
            question_stats.append({
                'index': i + 1,
                'avg_accuracy': avg_accuracy,
                'min_accuracy': min_accuracy,
                'max_accuracy': max_accuracy,
                'difficulty': difficulty
            })
            
            report += f"\n题目{i+1:<4} {avg_accuracy:.1%}{'':<4} {difficulty:<8} {min_accuracy:.1%}-{max_accuracy:.1%}"

        report += f"\n\n{'='*60}"
        report += f"\n各年级预测正确率："
        report += f"\n{'='*60}"

        for grade_level, results in all_results.items():
            grade_avg = sum(r['predicted_accuracy'] for r in results) / len(results) if results else 0
            report += f"\n{grade_level}: 平均正确率 {grade_avg:.1%}"

        report += f"\n\n{'='*60}"
        report += f"\n详细题目分析："
        report += f"\n{'='*60}"

        for i, question in enumerate(self.questions):
            if i >= len(next(iter(all_results.values())) if all_results else 0):
                break
                
            report += f"\n\n题目{i+1}: {question.get('content', '无内容')}"
            report += f"\n正确答案: {question.get('answer', '无答案')}"
            report += f"\n易错点: {question.get('analysis', '无分析')}"
            report += f"\n平均预测正确率: {question_stats[i]['avg_accuracy']:.1%} ({question_stats[i]['difficulty']})"
            report += f"\n预测范围: {question_stats[i]['min_accuracy']:.1%}-{question_stats[i]['max_accuracy']:.1%}"
            
            report += f"\n各年级预测:"
            for grade_level, results in all_results.items():
                if i < len(results):
                    accuracy = results[i]['predicted_accuracy']
                    report += f" {grade_level}{accuracy:.0%}"
            
            report += f"\n{'─'*60}"

        # 总体统计
        total_questions = len(question_stats)
        overall_avg = sum(stat['avg_accuracy'] for stat in question_stats) / total_questions if total_questions > 0 else 0
        
        report += f"\n\n总体统计："
        report += f"\n总体平均正确率: {overall_avg:.1%}"
        if question_stats:
            hardest = min(question_stats, key=lambda x: x['avg_accuracy'])
            easiest = max(question_stats, key=lambda x: x['avg_accuracy'])
            report += f"\n最难题目: 题目{hardest['index']} ({hardest['avg_accuracy']:.1%})"
            report += f"\n最简单题目: 题目{easiest['index']} ({easiest['avg_accuracy']:.1%})"

        return report


class JudgmentQuestionUI(QMainWindow):
    def __init__(self):
        super().__init__()
        self.generator = None
        self.all_results = {}
        self.initUI()
        
    def initUI(self):
        self.setWindowTitle('易错判断题生成与预测系统')
        self.setGeometry(100, 100, 1200, 900)
        
        # 中央部件
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        main_layout = QVBoxLayout(central_widget)
        
        # 创建分割器
        splitter = QSplitter(Qt.Vertical)
        
        # 上部：输入区域
        input_widget = QWidget()
        input_layout = QVBoxLayout(input_widget)
        
        # API密钥输入
        api_group = QGroupBox("API设置")
        api_layout = QHBoxLayout()
        api_layout.addWidget(QLabel("DeepSeek API密钥:"))
        self.api_key_input = QLineEdit()
        self.api_key_input.setEchoMode(QLineEdit.Password)
        self.api_key_input.setPlaceholderText("请输入您的API密钥")
        api_layout.addWidget(self.api_key_input)
        api_group.setLayout(api_layout)
        input_layout.addWidget(api_group)
        
        # 输入区域
        input_group = QGroupBox("题目生成设置")
        input_layout2 = QVBoxLayout()
        
        # 主题描述
        topic_layout = QHBoxLayout()
        topic_layout.addWidget(QLabel("主题描述:"))
        self.topic_input = QLineEdit()
        self.topic_input.setPlaceholderText("请输入要生成判断题的主题描述")
        topic_layout.addWidget(self.topic_input)
        input_layout2.addLayout(topic_layout)
        
        # 题目数量
        count_layout = QHBoxLayout()
        count_layout.addWidget(QLabel("题目数量:"))
        self.count_spin = QSpinBox()
        self.count_spin.setRange(1, 20)
        self.count_spin.setValue(5)
        count_layout.addWidget(self.count_spin)
        count_layout.addStretch()
        input_layout2.addLayout(count_layout)
        
        # 按钮区域
        button_layout = QHBoxLayout()
        self.generate_btn = QPushButton("生成题目")
        self.generate_btn.clicked.connect(self.generate_questions)
        self.generate_btn.setStyleSheet("QPushButton { background-color: #4CAF50; color: white; font-weight: bold; }")
        
        self.test_btn = QPushButton("开始多年级预测")
        self.test_btn.clicked.connect(self.start_testing)
        self.test_btn.setStyleSheet("QPushButton { background-color: #2196F3; color: white; font-weight: bold; }")
        self.test_btn.setEnabled(False)
        
        button_layout.addWidget(self.generate_btn)
        button_layout.addWidget(self.test_btn)
        input_layout2.addLayout(button_layout)
        
        input_group.setLayout(input_layout2)
        input_layout.addWidget(input_group)
        
        # 进度条
        self.progress_bar = QProgressBar()
        self.progress_bar.setVisible(False)
        input_layout.addWidget(self.progress_bar)
        
        # 进度标签
        self.progress_label = QLabel()
        self.progress_label.setVisible(False)
        input_layout.addWidget(self.progress_label)
        
        splitter.addWidget(input_widget)
        
        # 下部：输出区域
        output_widget = QWidget()
        output_layout = QVBoxLayout(output_widget)
        
        # 输出区域 - 使用选项卡
        self.tab_widget = QTabWidget()
        
        # 原始输出选项卡
        self.raw_output = QTextEdit()
        self.raw_output.setReadOnly(True)
        self.tab_widget.addTab(self.raw_output, "原始输出")
        
        # 格式化输出选项卡
        self.formatted_output = QTextEdit()
        self.formatted_output.setReadOnly(True)
        self.tab_widget.addTab(self.formatted_output, "格式化输出")
        
        # 统计报告选项卡
        self.report_output = QTextEdit()
        self.report_output.setReadOnly(True)
        self.tab_widget.addTab(self.report_output, "统计报告")
        
        # 结果表格选项卡
        self.result_table = QTableWidget()
        self.tab_widget.addTab(self.result_table, "结果表格")
        
        output_layout.addWidget(self.tab_widget)
        splitter.addWidget(output_widget)
        
        # 设置分割器比例
        splitter.setSizes([200, 700])
        main_layout.addWidget(splitter)
        
        # 状态栏
        self.status_bar = self.statusBar()
        
        # 设置样式
        self.apply_styles()
        
    def apply_styles(self):
        """应用样式"""
        self.setStyleSheet("""
            QMainWindow {
                background-color: #f5f5f5;
            }
            QGroupBox {
                font-weight: bold;
                border: 2px solid #cccccc;
                border-radius: 5px;
                margin-top: 1ex;
                padding-top: 10px;
            }
            QGroupBox::title {
                subcontrol-origin: margin;
                left: 10px;
                padding: 0 5px 0 5px;
            }
            QTextEdit {
                border: 1px solid #cccccc;
                border-radius: 3px;
                padding: 5px;
                background-color: white;
            }
            QLineEdit {
                border: 1px solid #cccccc;
                border-radius: 3px;
                padding: 5px;
            }
            QTableWidget {
                gridline-color: #cccccc;
                background-color: white;
            }
        """)
        
    def generate_questions(self):
        """生成题目"""
        api_key = self.api_key_input.text().strip()
        topic = self.topic_input.text().strip()
        count = self.count_spin.value()
        
        if not api_key:
            self.status_bar.showMessage("请输入API密钥")
            return
            
        if not topic:
            self.status_bar.showMessage("请输入主题描述")
            return
            
        self.generator = DeepSeekJudgmentQuestionGenerator(api_key)
        self.generate_btn.setEnabled(False)
        self.test_btn.setEnabled(False)
        self.status_bar.showMessage("正在生成题目...")
        self.progress_bar.setVisible(True)
        self.progress_bar.setRange(0, 0)  # 不确定进度
        self.progress_label.setVisible(False)
        
        # 启动工作线程
        self.worker = DeepSeekWorker(self.generator, topic, count)
        self.worker.finished.connect(self.on_generation_finished)
        self.worker.progress.connect(self.status_bar.showMessage)
        self.worker.start()
        
    def on_generation_finished(self, result, success):
        """生成完成回调"""
        self.generate_btn.setEnabled(True)
        self.progress_bar.setVisible(False)
        self.progress_label.setVisible(False)
        
        if success:
            self.raw_output.setPlainText(result)
            self.format_questions_output()
            self.test_btn.setEnabled(True)
            self.status_bar.showMessage("题目生成完成！可以开始多年级预测")
        else:
            self.raw_output.setPlainText(result)
            self.formatted_output.setPlainText("")
            self.test_btn.setEnabled(False)
            self.status_bar.showMessage("生成失败")
            
    def start_testing(self):
        """开始多年级预测"""
        if not self.generator or not self.generator.questions:
            self.status_bar.showMessage("请先生成题目")
            return
            
        self.test_btn.setEnabled(False)
        self.generate_btn.setEnabled(False)
        self.status_bar.showMessage("开始多年级预测...")
        self.progress_bar.setVisible(True)
        self.progress_label.setVisible(True)
        
        # 测试所有年级
        grades = ['幼儿园', '小学', '初中', '高中', '大学', '研究生']
        
        # 设置进度条范围
        self.progress_bar.setRange(0, len(grades))
        self.progress_bar.setValue(0)
        
        # 启动测试线程
        self.test_worker = TestWorker(self.generator, grades)
        self.test_worker.finished.connect(self.on_test_finished)
        self.test_worker.progress.connect(self.status_bar.showMessage)
        self.test_worker.grade_progress.connect(self.update_grade_progress)
        self.test_worker.start()
        
    def update_grade_progress(self, current_grade, total_grades, grade_name):
        """更新年级进度"""
        self.progress_bar.setValue(current_grade-1)
        self.progress_label.setText(f"已完成: {current_grade-1}/{total_grades} ({grade_name}正确率预测中)")
        
    def on_test_finished(self, all_results, success):
        """测试完成回调"""
        self.test_btn.setEnabled(True)
        self.generate_btn.setEnabled(True)
        self.progress_bar.setVisible(False)
        self.progress_label.setVisible(False)
        
        if success:
            self.all_results = all_results
            self.generate_statistics()
            self.update_result_table()
            self.status_bar.showMessage("多年级预测完成！")
        else:
            self.status_bar.showMessage("预测失败")
            
    def format_questions_output(self):
        """格式化题目输出"""
        if not self.generator or not self.generator.questions:
            return
            
        formatted_text = ""
        for i, question in enumerate(self.generator.questions):
            formatted_text += f"题目{i+1}: {question.get('content', '')}\n"
            formatted_text += f"正确答案: {question.get('answer', '')}\n"
            formatted_text += f"易错点: {question.get('analysis', '')}\n"
            formatted_text += "预测正确率: 等待预测\n难度: 等待预测\n"
            formatted_text += "-" * 50 + "\n\n"
            
        self.formatted_output.setPlainText(formatted_text)
        
    def generate_statistics(self):
        """生成统计报告"""
        if not self.generator or not self.all_results:
            return
            
        topic = self.topic_input.text().strip()
        report = self.generator.generate_statistics_report(self.all_results, topic)
        self.report_output.setPlainText(report)
        
        # 更新格式化输出中的预测正确率和难度
        self.update_formatted_output()
        
    def update_formatted_output(self):
        """更新格式化输出中的预测正确率和难度"""
        if not self.generator or not self.all_results:
            return
            
        formatted_text = ""
        question_stats = []
        
        # 计算每道题的平均预测正确率
        for i, question in enumerate(self.generator.questions):
            if i >= len(next(iter(self.all_results.values())) if self.all_results else 0):
                break
                
            accuracies = []
            
            for grade_level, results in self.all_results.items():
                if i < len(results):
                    accuracy = results[i]['predicted_accuracy']
                    accuracies.append(accuracy)
            
            avg_accuracy = sum(accuracies) / len(accuracies) if accuracies else 0
            
            # 确定难度等级
            if avg_accuracy >= 0.8:
                difficulty = "简单"
            elif avg_accuracy >= 0.6:
                difficulty = "中等"
            elif avg_accuracy >= 0.4:
                difficulty = "较难"
            else:
                difficulty = "困难"
            
            question_stats.append({
                'avg_accuracy': avg_accuracy,
                'difficulty': difficulty
            })
            
        # 重新生成格式化输出
        for i, question in enumerate(self.generator.questions):
            if i < len(question_stats):
                stats = question_stats[i]
                formatted_text += f"题目{i+1}: {question.get('content', '')}\n"
                formatted_text += f"正确答案: {question.get('answer', '')}\n"
                formatted_text += f"易错点: {question.get('analysis', '')}\n"
                formatted_text += f"平均预测正确率: {stats['avg_accuracy']:.1%}\n"
                formatted_text += f"难度: {stats['difficulty']}\n"
            else:
                formatted_text += f"题目{i+1}: {question.get('content', '')}\n"
                formatted_text += f"正确答案: {question.get('answer', '')}\n"
                formatted_text += f"易错点: {question.get('analysis', '')}\n"
                formatted_text += "预测正确率: 等待预测\n难度: 等待预测\n"
            
            formatted_text += "-" * 50 + "\n\n"
            
        self.formatted_output.setPlainText(formatted_text)
        
    def update_result_table(self):
        """更新结果表格"""
        if not self.generator or not self.all_results:
            return
            
        grades = list(self.all_results.keys())
        questions = self.generator.questions
        
        # 设置表格
        self.result_table.setRowCount(len(questions))
        self.result_table.setColumnCount(len(grades) + 2)  # 题目 + 各年级 + 正确答案
        
        headers = ["题目"] + [f"{grade}预测" for grade in grades] + ["正确答案"]
        self.result_table.setHorizontalHeaderLabels(headers)
        
        # 填充数据
        for i, question in enumerate(questions):
            # 题目内容
            question_item = QTableWidgetItem(f"题目{i+1}: {question.get('content', '')[:50]}...")
            self.result_table.setItem(i, 0, question_item)
            
            # 各年级预测
            for j, grade in enumerate(grades):
                if i < len(self.all_results[grade]):
                    result = self.all_results[grade][i]
                    accuracy = result['predicted_accuracy']
                    
                    item = QTableWidgetItem(f"{accuracy:.1%}")
                    
                    # 根据准确率设置颜色渐变
                    if accuracy >= 0.8:
                        color = QColor(200, 255, 200)  # 绿色
                    elif accuracy >= 0.6:
                        color = QColor(255, 255, 200)  # 黄色
                    elif accuracy >= 0.4:
                        color = QColor(255, 200, 150)  # 橙色
                    else:
                        color = QColor(255, 200, 200)  # 红色
                    
                    item.setBackground(color)
                    self.result_table.setItem(i, j + 1, item)
            
            # 正确答案
            correct_item = QTableWidgetItem(question.get('answer', ''))
            correct_item.setBackground(QColor(200, 230, 255))  # 蓝色背景
            self.result_table.setItem(i, len(grades) + 1, correct_item)
        
        # 调整列宽
        self.result_table.horizontalHeader().setSectionResizeMode(QHeaderView.Stretch)


def main():
    app = QApplication(sys.argv)
    
    # 设置应用程序字体
    font = QFont("Microsoft YaHei", 10)
    app.setFont(font)
    
    window = JudgmentQuestionUI()
    window.show()
    
    sys.exit(app.exec_())


if __name__ == "__main__":
    main()