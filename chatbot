import tkinter as tk
from tkinter import ttk
import threading
import time
from openai import OpenAI
import json
import os
from datetime import datetime
import traceback
import requests


API_KEY = os.environ.get("OPENAI_API_KEY") or "your-api-key-here"  # Replace with your actual API key if not using environment variable
client = OpenAI(api_key="OPENAI_API_KEY")

HISTORY_FILE = "chat_history.json"

HUMANIZER_API_URL = "URL"
HUMANIZER_USER_ID = "Humanizer_API_KEY"  

def create_rounded_button(parent, text, command=None):
    canvas = tk.Canvas(parent, width=220, height=40, bg="#171717", highlightthickness=0)
    canvas.pack(pady=10)
    radius = 12
    x0, y0, x1, y1 = 5, 5, 215, 40
    text_padx = 20
    fill_color = "#212121"
    active_fill = "#171717"

    def draw_button(color):
        canvas.delete("all")
        canvas.create_arc(x0, y0, x0+2*radius, y0+2*radius, start=90, extent=90, fill=color, outline=color)
        canvas.create_arc(x1-2*radius, y0, x1, y0+2*radius, start=0, extent=90, fill=color, outline=color)
        canvas.create_arc(x0, y1-2*radius, x0+2*radius, y1, start=180, extent=90, fill=color, outline=color)
        canvas.create_arc(x1-2*radius, y1-2*radius, x1, y1, start=270, extent=90, fill=color, outline=color)
        canvas.create_rectangle(x0+radius, y0, x1-radius, y1, fill=color, outline=color)
        canvas.create_rectangle(x0, y0+radius, x1, y1-radius, fill=color, outline=color)
        canvas.create_text(x0 + text_padx, (y0 + y1)//2, text=text, fill="white", font=("Helvetica", 10, "bold"), anchor="w")

    draw_button(fill_color)
    if command:
        canvas.bind("<Button-1>", lambda event: command())
    canvas.bind("<Enter>", lambda e: draw_button(active_fill))
    canvas.bind("<Leave>", lambda e: draw_button(fill_color))
    return canvas

class HumanizerWindow:
    def __init__(self, parent):
        self.window = tk.Toplevel(parent)
        self.window.title("Text Humanizer")
        self.window.geometry("1000x600")
        self.window.configure(bg="#1F202B")
        
        self.main_frame = ttk.Frame(self.window)
        self.main_frame.pack(fill=tk.BOTH, expand=True)
        
        self.sidebar = tk.Frame(self.main_frame, bg="#171717", width=50)
        self.sidebar.pack(side=tk.LEFT, fill=tk.Y)
        self.sidebar.pack_propagate(False)
        
        self.content_area = tk.Frame(self.main_frame, bg="#212121")
        self.content_area.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        self.panes = tk.PanedWindow(self.content_area, orient=tk.HORIZONTAL, bg="#444654", 
                                   sashwidth=4, sashrelief=tk.RAISED)
        self.panes.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        
        self.original_frame = tk.Frame(self.panes, bg="#212121", padx=10, pady=10)
        
        # Humanized text pane
        self.humanized_frame = tk.Frame(self.panes, bg="#212121", padx=10, pady=10)
        
        self.panes.add(self.original_frame, width=500)
        self.panes.add(self.humanized_frame, width=500)
        
        self.build_sidebar()
        self.build_input_area()
        self.build_output_area()
        
    def build_sidebar(self):
        label = tk.Label(self.sidebar, text="H", font=("Helvetica", 16, "bold"), 
                        fg="white", bg="#171717")
        label.pack(pady=10)
        
    def build_input_area(self):
        tk.Label(self.original_frame, text="Original Text", font=("Helvetica", 14, "bold"),
                fg="white", bg="#212121").pack(anchor="w", pady=(0, 10))
        
        settings_frame = tk.Frame(self.original_frame, bg="#212121")
        settings_frame.pack(fill=tk.X, pady=10)
        
        tk.Label(settings_frame, text="Tone:", fg="white", bg="#212121").pack(side=tk.LEFT, padx=(0, 5))
        self.tone_var = tk.StringVar(value="professional")
        tones = ["professional", "casual", "academic", "friendly", "formal"]
        tone_menu = ttk.Combobox(settings_frame, textvariable=self.tone_var, values=tones, width=12)
        tone_menu.pack(side=tk.LEFT, padx=(0, 15))
        
        tk.Label(settings_frame, text="Purpose:", fg="white", bg="#212121").pack(side=tk.LEFT, padx=(0, 5))
        self.purpose_var = tk.StringVar(value="blog")
        purposes = ["blog", "email", "research paper", "article", "social media"]
        purpose_menu = ttk.Combobox(settings_frame, textvariable=self.purpose_var, values=purposes, width=12)
        purpose_menu.pack(side=tk.LEFT)
        
        self.beast_mode_var = tk.BooleanVar(value=True)
        beast_mode_cb = tk.Checkbutton(settings_frame, text="Beast Mode", variable=self.beast_mode_var,
                                      bg="#212121", fg="white", selectcolor="#212121", 
                                      activebackground="#212121", activeforeground="white")
        beast_mode_cb.pack(side=tk.RIGHT)
        
        self.input_text = tk.Text(self.original_frame, bg="#2A2B32", fg="white", 
                                 font=("Helvetica", 11), padx=10, pady=10,
                                 insertbackground="white")
        self.input_text.pack(fill=tk.BOTH, expand=True, pady=10)
        
        button_frame = tk.Frame(self.original_frame, bg="#212121")
        button_frame.pack(fill=tk.X, pady=10)
        
        humanize_button = tk.Canvas(button_frame, width=120, height=35, bg="#212121", highlightthickness=0)
        humanize_button.pack(side=tk.RIGHT)
        
        def draw_humanize_button(color):
            humanize_button.delete("all")
            x0, y0, x1, y1 = 0, 0, 120, 35
            radius = 10
            humanize_button.create_arc(x0, y0, x0+2*radius, y0+2*radius, start=90, extent=90, fill=color, outline=color)
            humanize_button.create_arc(x1-2*radius, y0, x1, y0+2*radius, start=0, extent=90, fill=color, outline=color)
            humanize_button.create_arc(x0, y1-2*radius, x0+2*radius, y1, start=180, extent=90, fill=color, outline=color)
            humanize_button.create_arc(x1-2*radius, y1-2*radius, x1, y1, start=270, extent=90, fill=color, outline=color)
            humanize_button.create_rectangle(x0+radius, y0, x1-radius, y1, fill=color, outline=color)
            humanize_button.create_rectangle(x0, y0+radius, x1, y1-radius, fill=color, outline=color)
            humanize_button.create_text(60, 17, text="Humanize Text", fill="white", font=("Helvetica", 10, "bold"))
        
        draw_humanize_button("#212121")
        
        humanize_button.bind("<Enter>", lambda e: draw_humanize_button("#171717"))
        humanize_button.bind("<Leave>", lambda e: draw_humanize_button("#212121"))
        humanize_button.bind("<Button-1>", lambda e: self.humanize_text())
        
    def build_output_area(self):
        tk.Label(self.humanized_frame, text="Humanized Text", font=("Helvetica", 14, "bold"),
                fg="white", bg="#212121").pack(anchor="w", pady=(0, 10))
        
        self.status_label = tk.Label(self.humanized_frame, text="", fg="white", bg="#212121")
        self.status_label.pack(fill=tk.X, pady=5)
        
        self.output_text = tk.Text(self.humanized_frame, bg="#2A2B32", fg="white", 
                                  font=("Helvetica", 11), padx=10, pady=10,
                                  insertbackground="white", state=tk.DISABLED)
        self.output_text.pack(fill=tk.BOTH, expand=True, pady=10)
        
        button_frame = tk.Frame(self.humanized_frame, bg="#212121")
        button_frame.pack(fill=tk.X, pady=10)
        
        copy_button = tk.Canvas(button_frame, width=100, height=35, bg="#212121", highlightthickness=0)
        copy_button.pack(side=tk.RIGHT)
        
        def draw_copy_button(color):
            copy_button.delete("all")
            x0, y0, x1, y1 = 0, 0, 100, 35
            radius = 10

            copy_button.create_arc(x0, y0, x0+2*radius, y0+2*radius, start=90, extent=90, fill=color, outline=color)

            copy_button.create_arc(x1-2*radius, y0, x1, y0+2*radius, start=0, extent=90, fill=color, outline=color)

            copy_button.create_arc(x0, y1-2*radius, x0+2*radius, y1, start=180, extent=90, fill=color, outline=color)

            copy_button.create_arc(x1-2*radius, y1-2*radius, x1, y1, start=270, extent=90, fill=color, outline=color)

            copy_button.create_rectangle(x0+radius, y0, x1-radius, y1, fill=color, outline=color)
            copy_button.create_rectangle(x0, y0+radius, x1, y1-radius, fill=color, outline=color)

            copy_button.create_text(50, 17, text="Copy Text", fill="white", font=("Helvetica", 10, "bold"))
        
        draw_copy_button("#212121")
        
        copy_button.bind("<Enter>", lambda e: draw_copy_button("#171717"))
        copy_button.bind("<Leave>", lambda e: draw_copy_button("#212121"))
        copy_button.bind("<Button-1>", lambda e: self.copy_to_clipboard())
        
    def humanize_text(self):
        input_content = self.input_text.get("1.0", tk.END).strip()
        if not input_content:
            self.status_label.config(text="Please enter some text to humanize")
            return
            

        self.status_label.config(text="Humanizing text...")
        

        data = {
            "text": input_content,
            "tone": self.tone_var.get(),
            "purpose": self.purpose_var.get(),
            "language": "english",
            "beast_mode": self.beast_mode_var.get(),
            "shouldStream": False,
            "user_id": HUMANIZER_USER_ID
        }
        

        threading.Thread(target=self.run_humanization, args=(data,)).start()
        
    def run_humanization(self, data):
        try:

            response = requests.post(HUMANIZER_API_URL, json=data)
            
            if response.status_code == 200:
                result = response.json()
                humanized_text = result.get("content", "No content returned")
                

                self.window.after(0, lambda: self.update_output(humanized_text))
            else:
                error_msg = f"API Error: {response.status_code} - {response.text}"
                self.window.after(0, lambda: self.show_error(error_msg))
                
        except Exception as e:
            error_msg = f"Error: {str(e)}"
            self.window.after(0, lambda: self.show_error(error_msg))
            
    def update_output(self, text):
        self.output_text.config(state=tk.NORMAL)
        self.output_text.delete("1.0", tk.END)
        self.output_text.insert("1.0", text)
        self.output_text.config(state=tk.DISABLED)
        self.status_label.config(text="Humanization complete!")
        
    def show_error(self, error_msg):
        self.status_label.config(text=error_msg)
        
    def copy_to_clipboard(self):
        humanized_text = self.output_text.get("1.0", tk.END).strip()
        self.window.clipboard_clear()
        self.window.clipboard_append(humanized_text)
        self.status_label.config(text="Copied to clipboard!")
        
class ChatGPTClone:
    def __init__(self, root):
        self.root = root
        self.root.title("NiggaGPT")
        self.root.geometry("1200x800")
        self.root.configure(bg="#1F202B")
        self.main_frame = ttk.Frame(self.root)
        self.main_frame.pack(fill=tk.BOTH, expand=True)

        self.sidebar = tk.Frame(self.main_frame, bg="#171717", width=260)
        self.sidebar.pack(side=tk.LEFT, fill=tk.Y)
        self.sidebar.pack_propagate(False)  

        self.chat_area = tk.Frame(self.main_frame, bg="#212121")
        self.chat_area.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

        self.current_chat_id = None
        
        self.chat_history = {}
        
        self.current_messages = []
        
        self.load_history()
        
        self.build_sidebar()
        self.build_chat_area()
        
        self.new_chat()

    def load_history(self):
        if os.path.exists(HISTORY_FILE):
            try:
                with open(HISTORY_FILE, 'r', encoding='utf-8') as f:
                    loaded_data = json.load(f)
                    if isinstance(loaded_data, dict):
                        self.chat_history = loaded_data
                    else:
                        self.chat_history = {}
            except Exception as e:
                print(f"Error loading chat history: {e}")
                self.chat_history = {}
        else:
            self.chat_history = {}

    def save_history(self):
        try:
            with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.chat_history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Error saving chat history: {e}")

    def build_sidebar(self):
        create_rounded_button(self.sidebar, "+  New Chat", command=self.new_chat)
        
        divider = tk.Frame(self.sidebar, height=1, bg="#444654")
        divider.pack(fill=tk.X, pady=10, padx=10)
        
        self.history_container = tk.Frame(self.sidebar, bg="#171717")
        self.history_container.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)
        
        self.history_canvas = tk.Canvas(self.history_container, bg="#171717", highlightthickness=0)
        self.history_scrollbar = ttk.Scrollbar(self.history_container, orient="vertical", command=self.history_canvas.yview)
        self.history_frame = tk.Frame(self.history_canvas, bg="#171717")
        
        self.history_canvas.configure(yscrollcommand=self.history_scrollbar.set)
        self.history_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.history_canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        self.history_canvas_frame = self.history_canvas.create_window((0, 0), window=self.history_frame, anchor="nw")
        self.history_frame.bind("<Configure>", lambda e: self.history_canvas.configure(scrollregion=self.history_canvas.bbox("all")))
        self.history_canvas.bind("<Configure>", self.on_history_canvas_configure)
        
        self.update_history_display()
    
    def on_history_canvas_configure(self, event):
        width = event.width
        self.history_canvas.itemconfig(self.history_canvas_frame, width=width)

    def update_history_display(self):
        for widget in self.history_frame.winfo_children():
            widget.destroy()
            
        if isinstance(self.chat_history, dict):
            for chat_id, chat_data in sorted(self.chat_history.items(), 
                                            key=lambda x: x[1].get('timestamp', 0), 
                                            reverse=True):
                self.create_history_item(chat_id, chat_data)

    def create_history_item(self, chat_id, chat_data):
        item_frame = tk.Frame(self.history_frame, bg="#171717")
        item_frame.pack(fill=tk.X, pady=2)
        
        title = chat_data.get('title', 'New conversation')
        if len(title) > 23:
            title = title[:20] + "..."
        
        chat_button = tk.Canvas(item_frame, width=180, height=30, bg="#171717", highlightthickness=0)
        chat_button.pack(side=tk.LEFT)
        
        radius = 8
        fill_color = "#212121" if chat_id == self.current_chat_id else "#171717"
        active_fill = "#171717"  
        
        def draw_rounded_button(color):
            chat_button.delete("all")
            x0, y0, x1, y1 = 0, 0, 180, 30
            r = radius
            
            chat_button.create_arc(x0, y0, x0+2*r, y0+2*r, start=90, extent=90, fill=color, outline=color)
            chat_button.create_arc(x1-2*r, y0, x1, y0+2*r, start=0, extent=90, fill=color, outline=color)
            chat_button.create_arc(x0, y1-2*r, x0+2*r, y1, start=180, extent=90, fill=color, outline=color)
            chat_button.create_arc(x1-2*r, y1-2*r, x1, y1, start=270, extent=90, fill=color, outline=color)
            chat_button.create_rectangle(x0+r, y0, x1-r, y1, fill=color, outline=color)
            chat_button.create_rectangle(x0, y0+r, x1, y1-r, fill=color, outline=color)
            
            chat_button.create_text(10, 15, text=title, fill="white", font=("Helvetica", 9), anchor="w")
        
        draw_rounded_button(fill_color)
        
        def on_enter(e):
            if chat_id == self.current_chat_id:
                draw_rounded_button("#252525")
            else:
                draw_rounded_button(active_fill)
                
        def on_leave(e):
            if chat_id == self.current_chat_id:
                draw_rounded_button("#212121")
            else:
                draw_rounded_button("#171717")
                
        def on_click(e):
            self.load_chat(chat_id)
            
        chat_button.bind("<Enter>", on_enter)
        chat_button.bind("<Leave>", on_leave)
        chat_button.bind("<Button-1>", on_click)
        
        # Delete button
        delete_btn = tk.Canvas(item_frame, width=20, height=20, bg="#171717", highlightthickness=0)
        delete_btn.pack(side=tk.RIGHT, padx=5)
        
        def draw_delete_btn(color):
            delete_btn.delete("all")
            delete_btn.create_oval(0, 0, 20, 20, fill=color, outline=color)
            delete_btn.create_text(10, 10, text="×", fill="white", font=("Helvetica", 12, "bold"))
            
        draw_delete_btn("#171717")
        
        def delete_on_enter(e):
            draw_delete_btn("#444654")
            
        def delete_on_leave(e):
            draw_delete_btn("#171717")
            
        def delete_on_click(e):
            if chat_id in self.chat_history:
                del self.chat_history[chat_id]
                self.save_history()
                self.update_history_display()
                if chat_id == self.current_chat_id:
                    self.new_chat()
                    
        delete_btn.bind("<Enter>", delete_on_enter)
        delete_btn.bind("<Leave>", delete_on_leave)
        delete_btn.bind("<Button-1>", delete_on_click)

    def build_chat_area(self):
        self.messages_canvas_frame = tk.Frame(self.chat_area, bg="#212121")
        self.messages_canvas_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=(20, 10))
        
        self.messages_canvas = tk.Canvas(self.messages_canvas_frame, bg="#212121", highlightthickness=0)
        self.messages_scrollbar = ttk.Scrollbar(self.messages_canvas_frame, orient="vertical", command=self.messages_canvas.yview)
        
        self.message_container = tk.Frame(self.messages_canvas, bg="#212121")
        self.messages_window = self.messages_canvas.create_window((0, 0), window=self.message_container, anchor="nw")
        
        self.messages_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.messages_canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        self.message_container.bind("<Configure>", lambda e: self.messages_canvas.configure(scrollregion=self.messages_canvas.bbox("all")))
        self.messages_canvas.bind("<Configure>", self.on_messages_canvas_configure)

        input_container = tk.Frame(self.chat_area, bg="#212121")
        input_container.pack(pady=(0, 20))

        self.input_bg = "#2A2B32"
        self.input_radius = 25
        input_width = 600

        self.input_box = tk.Canvas(input_container, width=input_width, height=50, bg="#212121", highlightthickness=0)
        self.input_box.pack(side=tk.LEFT)

        def draw_input_box():
            self.input_box.delete("all")
            w = self.input_box.winfo_width()
            r = self.input_radius
            self.input_box.create_arc(0, 0, 2*r, 2*r, start=90, extent=90, fill=self.input_bg, outline=self.input_bg)
            self.input_box.create_arc(w-2*r, 0, w, 2*r, start=0, extent=90, fill=self.input_bg, outline=self.input_bg)
            self.input_box.create_arc(0, 50-2*r, 2*r, 50, start=180, extent=90, fill=self.input_bg, outline=self.input_bg)
            self.input_box.create_arc(w-2*r, 50-2*r, w, 50, start=270, extent=90, fill=self.input_bg, outline=self.input_bg)
            self.input_box.create_rectangle(r, 0, w-r, 50, fill=self.input_bg, outline=self.input_bg)
            self.input_box.create_rectangle(0, r, w, 50-r, fill=self.input_bg, outline=self.input_bg)

        self.input_box.bind("<Configure>", lambda e: draw_input_box())

        self.entry = tk.Entry(self.input_box, font=("Helvetica", 12), bg=self.input_bg, fg="white", bd=0,
                              insertbackground="white", relief=tk.FLAT)
        self.entry.place(x=15, y=13, relwidth=0.87, height=25)
        
        self.entry.bind("<Return>", lambda e: self.send_message())

        send_canvas = tk.Canvas(self.input_box, width=40, height=40, bg=self.input_bg, highlightthickness=0)
        send_canvas.place(relx=1.0, x=-50, y=5)

        def draw_send_button(color):
            send_canvas.delete("all")
            send_canvas.create_oval(0, 0, 40, 40, fill=color, outline=color)
            send_canvas.create_text(20, 20, text="↑", fill="white", font=("Helvetica", 14, "bold"))

        normal_color = "#40414F"
        hover_color = "#35363C"
        draw_send_button(normal_color)
        send_canvas.bind("<Enter>", lambda e: draw_send_button(hover_color))
        send_canvas.bind("<Leave>", lambda e: draw_send_button(normal_color))
        send_canvas.bind("<Button-1>", lambda e: self.send_message())
        
        humanizer_button = tk.Canvas(input_container, width=100, height=50, bg="#212121", highlightthickness=0)
        humanizer_button.pack(side=tk.LEFT, padx=10)
        
        def draw_humanizer_button(color):
            humanizer_button.delete("all")
            radius = 15
            x0, y0, x1, y1 = 0, 5, 100, 45  
            humanizer_button.create_arc(x0, y0, x0+2*radius, y0+2*radius, start=90, extent=90, fill=color, outline=color)
            humanizer_button.create_arc(x1-2*radius, y0, x1, y0+2*radius, start=0, extent=90, fill=color, outline=color)
            humanizer_button.create_arc(x0, y1-2*radius, x0+2*radius, y1, start=180, extent=90, fill=color, outline=color)
            humanizer_button.create_arc(x1-2*radius, y1-2*radius, x1, y1, start=270, extent=90, fill=color, outline=color)
            humanizer_button.create_rectangle(x0+radius, y0, x1-radius, y1, fill=color, outline=color)
            humanizer_button.create_rectangle(x0, y0+radius, x1, y1-radius, fill=color, outline=color)
            
            humanizer_button.create_text(50, 25, text="Humanizer", fill="white", font=("Helvetica", 10, "bold"))
        
        draw_humanizer_button("#40414F")
        
        humanizer_button.bind("<Enter>", lambda e: draw_humanizer_button("#35363C"))
        humanizer_button.bind("<Leave>", lambda e: draw_humanizer_button("#40414F"))
        humanizer_button.bind("<Button-1>", lambda e: self.open_humanizer())

    def on_messages_canvas_configure(self, event):
        width = event.width
        self.messages_canvas.itemconfig(self.messages_window, width=width)
        
    def open_humanizer(self):
        humanizer = HumanizerWindow(self.root)

    def new_chat(self):
        chat_id = str(int(time.time()))
        
        self.chat_history[chat_id] = {
            'title': 'New conversation',
            'timestamp': int(time.time()),
            'messages': []
        }
        
        self.save_history()
        
        self.update_history_display()
        
        self.load_chat(chat_id)

    def load_chat(self, chat_id):
        self.current_chat_id = chat_id
        
        for widget in self.message_container.winfo_children():
            widget.destroy()
            
        if chat_id in self.chat_history:
            self.current_messages = self.chat_history[chat_id].get('messages', [])
            
            for message in self.current_messages:
                self.display_message(message['role'], message['content'])
        else:
            self.current_messages = []
            
        self.update_history_display()

    def send_message(self):
        message = self.entry.get().strip()
        if not message:
            return
            
        self.entry.delete(0, tk.END)
        
        self.display_message("user", message)
        
        self.current_messages.append({
            'role': 'user',
            'content': message
        })
        
        if len(self.current_messages) == 1 and self.current_chat_id in self.chat_history:
            self.chat_history[self.current_chat_id]['title'] = message
            self.update_history_display()
        
        if self.current_chat_id in self.chat_history:
            self.chat_history[self.current_chat_id]['messages'] = self.current_messages
            self.save_history()
        
        loading_frame = self.display_loading_indicator()
        
        threading.Thread(target=self.get_ai_response, args=(message, loading_frame)).start()

    def get_ai_response(self, message, loading_frame):
        try:
            messages = [{"role": msg["role"], "content": msg["content"]} for msg in self.current_messages]
            
            response = client.chat.completions.create(
              model="gpt-3.5-turbo",  
              messages=messages,
            )
            
            ai_message = response.choices[0].message.content
            
            self.current_messages.append({
                'role': 'assistant',
                'content': ai_message
            })
            
            if self.current_chat_id in self.chat_history:
                self.chat_history[self.current_chat_id]['messages'] = self.current_messages
                self.save_history()
            
            self.root.after(0, lambda: self.update_ui_with_response(ai_message, loading_frame))
            
        except Exception as e:
            error_message = f"Error: {str(e)}\n{traceback.format_exc()}"
            print(error_message)
            self.root.after(0, lambda: self.update_ui_with_response(f"I encountered an error while processing your request: {str(e)}", loading_frame))

    def update_ui_with_response(self, response_text, loading_frame):
        if loading_frame and loading_frame.winfo_exists():
            loading_frame.destroy()
            
        self.display_message("assistant", response_text)
        
        self.messages_canvas.update_idletasks()
        self.messages_canvas.yview_moveto(1.0)

    def display_loading_indicator(self):
        loading_frame = tk.Frame(self.message_container, bg="#444654", padx=20, pady=10)
        loading_frame.pack(fill=tk.X, pady=5)
        
        loading_label = tk.Label(loading_frame, text="ChatGPT is thinking...", fg="white", bg="#444654", font=("Helvetica", 10))
        loading_label.pack(anchor="w", pady=5)
        
        dots_label = tk.Label(loading_frame, text="", fg="white", bg="#444654", font=("Helvetica", 10))
        dots_label.pack(anchor="w")
        
        def animate_dots(count=0):
            dots = "." * (count % 4)
            dots_label.config(text=dots)
            if loading_frame.winfo_exists():
                self.root.after(500, animate_dots, count + 1)
                
        animate_dots()
        
        self.message_container.update_idletasks()
        self.messages_canvas.yview_moveto(1.0)
        
        return loading_frame

    def display_message(self, role, content):
        is_user = (role == "user")
    
        message_frame = tk.Frame(self.message_container, bg="#212121")
        message_frame.pack(fill=tk.X, pady=0)
    
        bubble_canvas = tk.Canvas(message_frame, bg="#212121", highlightthickness=0)
        text_padding = 20
        max_width = 700

        text_id = bubble_canvas.create_text(text_padding, text_padding, anchor='nw', text=content,
                                      fill="white", font=("Helvetica", 11), width=max_width - 2 * text_padding)

        bbox = bubble_canvas.bbox(text_id)
        width = min(max_width, bbox[2] + text_padding)
        height = bbox[3] + text_padding

        bg_color = "#2A2B32" if is_user else "#343541"
        bubble_canvas.delete("all")

        r = 12  
        bubble_canvas.config(width=width, height=height)

        bubble_canvas.create_arc(0, 0, 2*r, 2*r, start=90, extent=90, fill=bg_color, outline=bg_color)
        bubble_canvas.create_arc(width-2*r, 0, width, 2*r, start=0, extent=90, fill=bg_color, outline=bg_color)
        bubble_canvas.create_arc(0, height-2*r, 2*r, height, start=180, extent=90, fill=bg_color, outline=bg_color)
        bubble_canvas.create_arc(width-2*r, height-2*r, width, height, start=270, extent=90, fill=bg_color, outline=bg_color)
        bubble_canvas.create_rectangle(r, 0, width-r, height, fill=bg_color, outline=bg_color)
        bubble_canvas.create_rectangle(0, r, width, height-r, fill=bg_color, outline=bg_color)

        bubble_canvas.create_text(text_padding, text_padding, anchor='nw', text=content,
                            fill="white", font=("Helvetica", 11), width=max_width - 2 * text_padding)

        side_pad = (100, 0) if not is_user else (0, 100)
        anchor = "e" if is_user else "w"
        bubble_canvas.pack(padx=side_pad, pady=8, anchor=anchor)
    
        self.message_container.update_idletasks()
        self.messages_canvas.yview_moveto(1.0)
def main():
    root = tk.Tk()
    app = ChatGPTClone(root)
    root.mainloop()

if __name__ == "__main__":
    main()
