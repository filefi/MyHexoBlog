---
title: tkinter 一例
date: 2022-02-16 23:56:21
tags: [Python, tkinter, GUI]
categories: Python
---


```python
import tkinter as tk
import tkinter.ttk as ttk
import subprocess
import shlex
import sys


class App(tk.Frame):
	def __init__(self,master=None):
		super().__init__(master)
		self.pack(fill=tk.BOTH)
		self.create_widgets()
	
	def test(self):
		content = self.entry.get()
		if content:
			cmd = shlex.split(content)
			result = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, encoding="gbk")
			# 实时获取子进程输出的每一行内容
			for line in iter(result.stdout.readline, ''):
				# 将子进程输出的每一行写入text文本框
				self.text.insert(tk.END, line)
				# 立即重绘Text内容
				self.text.update()
			
	
	def create_widgets(self):
		self.entry = ttk.Entry(self)
		self.entry.grid(row=0, column=0, sticky=tk.W+tk.E)
		
		
		self.label_split = ttk.Label(self)
		self.label_split.grid(row=0, column=1)
		
		self.button = ttk.Button(self, text="确定", width=10, command=self.test)
		self.button.grid(row=0, column=2, sticky=tk.E)
		
		self.label_split = ttk.Label(self)
		self.label_split.grid(row=1, column=0, sticky=tk.W+tk.E)
		
		self.scrollbar = ttk.Scrollbar(self)
		self.text = tk.Text(self, yscrollcommand=self.scrollbar.set)
		# 渲染以上2个组件
		self.text.grid(row=2, column=0, sticky=tk.W+tk.E+tk.N+tk.S)
		self.scrollbar.grid(row=2, column=1,sticky=tk.N+tk.S)
		self.scrollbar.config(command=self.text.yview)
		
		
def main():
	root = tk.Tk()
	root.title = "Test"
	
	app = App(master=root)
	app.mainloop()


if __name__ == "__main__":
	main()

```



