# MATRIX
MATRIX cmd
pip install pyinstaller
import random
import sys
import json
import os
import subprocess
import tkinter as tk
from tkinter import ttk, colorchooser, messagebox

if sys.platform.startswith("win"):
    import ctypes

APPDATA_DIR = os.getenv("APPDATA") or os.getenv("TEMP") or os.path.expanduser("~")
CONFIG_FILE = os.path.join(APPDATA_DIR, "matrix_config.json")

LANG = {
    "ar": {
        "title": "إعدادات شاشة ماتريكس",
        "btn_lang": "English 🌐",
        "term_title": "عنوان النافذة المخصص:",
        "disp_mode": "وضع العرض:",
        "normal_win": "نافذة عادية",
        "full_win": "ملء الشاشة (Fullscreen)",
        "always_on_top": "تثبيت النافذة فوق كل النوافذ دائماً (Always on Top)",
        "frameless": "إزالة شريط العنوان (نافذة بدون إطار قابلة للسحب)",
        "fade_style": "نمط الاختفاء:",
        "fade_opt1": "تلاشي تدريجي أثناء الهبوط (Fade Out)",
        "fade_opt2": "الهبوط الكامل حتى أسفل الشاشة (Vanish Out of Sight)",
        "fade_opt3": "تلاشي تدريجي من الأسفل عند النزول (Bottom Fade Out)",
        "bg_trans": "لون الخلفية والشفافية:",
        "pick_bg": "اختر لون خلفية مخصص",
        "reset": "إعادة ضبط",
        "pools": "مجموعات الحروف (يمكنك الدمج بينها):",
        "pool_bin": "ثنائي (0,1)",
        "pool_hex": "ست عشري (0-9,A-F)",
        "pool_eng": "إنجليزي (A-Z)",
        "pool_arab": "عربي (أ-ي)",
        "pool_kata": "كاتاكانا (Katakana)",
        "pool_sym": "رموز (!@#$)",
        "custom_chars": "حروف / كلمات مخصصة إضافية:",
        "text_color": "تحديد لون النصوص:",
        "pick_fg": "اختر لون نص مخصص",
        "adv_settings": "إعدادات التخصيص المتقدمة:",
        "speed": "السرعة (ms):",
        "font_size": "حجم الخط:",
        "tail_len": "طول الذيل الأساسي:",
        "win_alpha": "شفافية النافذة العامة:",
        "bg_alpha_toggle": "شفافية الخلفية (صفرية / طبيعية):",
        "text_alpha": "شفافية الحروف والرموز:",
        "white_head": "تفعيل لون مخصص للحرف الأول القائد",
        "pick_head": "اختر لون الحرف القائد (#FFFFFF)",
        "reset_head": "إعادة ضبط للأبيض",
        "tail_tip": "تفعيل لون مميز لأخر حرف في الذيل",
        "dynamic_shuffle": "تغيير وتشفير الحروف باستمرار أثناء الهبوط",
        "launch": "تشغيل ماتريكس",
        "save": "حفظ الإعدادات",
        "back_btn": "⬅ العودة للإعدادات",
        "warn_title": "تحذير توافقية الشفافية (Linux)",
        "warn_msg": "الشفافية تطلب وجود Compositor مشغل على النظام.\nهل تريد أن يحاول السكربت تثبيت 'picom' تلقائياً؟",
        "success_save": "تم حفظ إعدادات الماتريكس بنجاح!",
        "err_save": "تعذر حفظ الملف:"
    },
    "en": {
        "title": "MATRIX TERMINAL CONFIG",
        "btn_lang": "عربي 🌐",
        "term_title": "Terminal Custom Title:",
        "disp_mode": "Display Mode:",
        "normal_win": "Normal Window",
        "full_win": "Fullscreen Mode",
        "always_on_top": "Keep Window Always on Top",
        "frameless": "Remove Title Bar (Frameless & Draggable Window)",
        "fade_style": "Disappear Style:",
        "fade_opt1": "Fade out gradually while falling",
        "fade_opt2": "Go all the way down and vanish out of sight",
        "fade_opt3": "Gradually fade out from the bottom",
        "bg_trans": "Background Color & Transparency:",
        "pick_bg": "Pick Custom BG Color",
        "reset": "Reset",
        "pools": "Character Pools (Combine as you like):",
        "pool_bin": "Binary (0,1)",
        "pool_hex": "Hex (0-9,A-F)",
        "pool_eng": "English (A-Z)",
        "pool_arab": "Arabic (أ-ي)",
        "pool_kata": "Katakana",
        "pool_sym": "Symbols (!@#)",
        "custom_chars": "Custom Typed Characters / Words:",
        "text_color": "Text Color Selection:",
        "pick_fg": "Pick Custom Text Color",
        "adv_settings": "Advanced Customization Settings:",
        "speed": "Speed (ms):",
        "font_size": "Font Size:",
        "tail_len": "Tail Length (Base):",
        "win_alpha": "Window Alpha:",
        "bg_alpha_toggle": "Background Alpha (Zero / Normal):",
        "text_alpha": "Letters Alpha:",
        "white_head": "Enable custom color for the leading top character",
        "pick_head": "Pick Leading Char Color (#FFFFFF)",
        "reset_head": "Reset to White",
        "tail_tip": "Enable distinct color for the last tail character",
        "dynamic_shuffle": "Continuously shuffle characters as they fall",
        "launch": "LAUNCH CONFIG",
        "save": "SAVE CONFIG",
        "back_btn": "⬅ Back to Config",
        "warn_title": "Transparency Compatibility Warning",
        "warn_msg": "Transparency requires a Compositor running on Linux.\nWould you like the script to attempt installing 'picom'?",
        "success_save": "Matrix configuration saved successfully!",
        "err_save": "Could not save file:"
    }
}

def check_system_compatibility():
    if sys.platform.startswith("linux"):
        try:
            subprocess.run(["pgrep", "-x", "picom"], check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
            return True
        except subprocess.CalledProcessError:
            try:
                subprocess.run(["pgrep", "-x", "xcompmgr"], check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
                return True
            except subprocess.CalledProcessError:
                return False
    return True

def attempt_install_compositor():
    try:
        subprocess.run(["sudo", "apt", "update"], check=True)
        subprocess.run(["sudo", "apt", "install", "-y", "picom"], check=True)
        messagebox.showinfo("Success", "Compositor installed. Please restart the script.")
    except Exception as e:
        messagebox.showerror("Error", f"Failed to install automatically: {e}\nPlease install 'picom' manually.")

def hex_to_rgb(hex_str):
    hex_str = hex_str.lstrip('#')
    return tuple(int(hex_str[i:i+2], 16) for i in (0, 2, 4))

def rgb_to_hex(rgb):
    return f"#{rgb[0]:02x}{rgb[1]:02x}{rgb[2]:02x}"

class LanguageSelector:
    def __init__(self, root):
        self.root = root
        self.selected_lang = "ar"
        self.root.title("Language Selection / اختيار اللغة")
        self.root.geometry("360x200")
        self.root.configure(bg="#111111")
        self.root.resizable(False, False)
        self.root.eval('tk::PlaceWindow . center')

        tk.Label(
            self.root, 
            text="Choose Configuration Language\nاختر لغة الإعدادات", 
            fg="#00FF00", bg="#111111", 
            font=("Consolas", 11, "bold")
        ).pack(pady=20)

        btn_frame = tk.Frame(self.root, bg="#111111")
        btn_frame.pack(pady=10)

        btn_ar = tk.Button(
            btn_frame, text="العربية", 
            command=lambda: self.select_language("ar"), 
            bg="#222222", fg="#00FF00", 
            font=("Consolas", 10, "bold"), width=12, relief="flat", bd=2
        )
        btn_ar.pack(side=tk.LEFT, padx=10)

        btn_en = tk.Button(
            btn_frame, text="English", 
            command=lambda: self.select_language("en"), 
            bg="#222222", fg="#00FF00", 
            font=("Consolas", 10, "bold"), width=12, relief="flat", bd=2
        )
        btn_en.pack(side=tk.RIGHT, padx=10)

    def select_language(self, lang):
        self.selected_lang = lang
        cfg = {}
        if os.path.exists(CONFIG_FILE):
            try:
                with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                    cfg = json.load(f)
            except Exception:
                pass
        cfg["lang"] = lang
        try:
            with open(CONFIG_FILE, "w", encoding="utf-8") as f:
                json.dump(cfg, f, indent=4)
        except Exception:
            pass
        self.root.destroy()

class MatrixLauncher:
    def __init__(self, root, selected_lang="ar"):
        self.root = root
        self.current_lang = selected_lang
        self.root.title("Matrix Engine Setup")
        self.root.geometry("580x850")
        self.root.configure(bg="#111111")
        self.root.minsize(500, 500)

        self.custom_bg = None
        self.custom_fg = None
        self.custom_head_color = None

        if not check_system_compatibility():
            install = messagebox.askyesno(LANG[self.current_lang]["warn_title"], LANG[self.current_lang]["warn_msg"])
            if install:
                attempt_install_compositor()

        self.main_container = tk.Frame(self.root, bg="#111111")
        self.main_container.pack(fill=tk.BOTH, expand=True)

        self.canvas = tk.Canvas(self.main_container, bg="#111111", highlightthickness=0)
        self.scrollbar = ttk.Scrollbar(self.main_container, orient=tk.VERTICAL, command=self.canvas.yview)
        
        self.scrollable_frame = tk.Frame(self.canvas, bg="#111111")
        self.scrollable_frame.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        
        self.canvas.create_window((0, 0), window=self.scrollable_frame, anchor="nw")
        self.canvas.configure(yscrollcommand=self.scrollbar.set)

        self.canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        self.scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.root.bind_all("<MouseWheel>", self._on_mousewheel)
        self.root.bind_all("<Button-4>", lambda e: self.canvas.yview_scroll(-1, "units"))
        self.root.bind_all("<Button-5>", lambda e: self.canvas.yview_scroll(1, "units"))

        self.fullscreen_var = tk.BooleanVar(value=False)
        self.always_on_top_var = tk.BooleanVar(value=False)
        self.frameless_var = tk.BooleanVar(value=False)
        self.fade_mode_var = tk.StringVar(value="out_of_sight")
        self.bg_combo_var = tk.StringVar(value="#000000 - Classic Black")
        self.color_combo_var = tk.StringVar(value="#00FF00 - Classic Green")
        
        self.pool_bin = tk.BooleanVar(value=True)
        self.pool_hex = tk.BooleanVar(value=True)
        self.pool_eng = tk.BooleanVar(value=True)
        self.pool_arab = tk.BooleanVar(value=False)
        self.pool_kata = tk.BooleanVar(value=True)
        self.pool_sym = tk.BooleanVar(value=True)

        self.white_head_var = tk.BooleanVar(value=True)
        self.custom_tail_tip_var = tk.BooleanVar(value=False)
        self.dynamic_shuffle_var = tk.BooleanVar(value=True)
        
        self.bg_alpha_state = tk.BooleanVar(value=False)

        self.build_ui()
        self.load_config_from_file()

    def _on_mousewheel(self, event):
        if event.delta:
            self.canvas.yview_scroll(int(-1 * (event.delta / 120)), "units")

    def toggle_lang(self):
        self.current_lang = "en" if self.current_lang == "ar" else "ar"
        cfg = self.get_config_dict()
        cfg["lang"] = self.current_lang
        try:
            with open(CONFIG_FILE, "w", encoding="utf-8") as f:
                json.dump(cfg, f, indent=4)
        except Exception:
            pass
        for widget in self.scrollable_frame.winfo_children():
            widget.destroy()
        self.build_ui()

    def build_ui(self):
        txt = LANG[self.current_lang]
        
        lang_btn = tk.Button(self.scrollable_frame, text=txt["btn_lang"], command=self.toggle_lang, bg="#222222", fg="#00FF00", font=("Consolas", 9, "bold"), relief="flat", bd=1)
        lang_btn.pack(anchor="e", padx=20, pady=5)

        tk.Label(self.scrollable_frame, text=txt["title"], fg="#00FF00", bg="#111111", font=("Consolas", 13, "bold")).pack(pady=6)

        tk.Label(self.scrollable_frame, text=txt["term_title"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30)
        self.terminal_title_entry = tk.Entry(self.scrollable_frame, width=44, font=("Consolas", 9))
        self.terminal_title_entry.insert(0, "Matrix Engine")
        self.terminal_title_entry.pack(anchor="w", padx=50, pady=2)

        tk.Label(self.scrollable_frame, text=txt["disp_mode"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(4, 0))
        tk.Radiobutton(self.scrollable_frame, text=txt["normal_win"], variable=self.fullscreen_var, value=False, fg="#00FF00", bg="#111111", selectcolor="#222222", font=("Consolas", 9)).pack(anchor="w", padx=50)
        tk.Radiobutton(self.scrollable_frame, text=txt["full_win"], variable=self.fullscreen_var, value=True, fg="#00FF00", bg="#111111", selectcolor="#222222", font=("Consolas", 9)).pack(anchor="w", padx=50)

        tk.Checkbutton(self.scrollable_frame, text=txt["always_on_top"], variable=self.always_on_top_var, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 9, "bold"), activebackground="#111111", activeforeground="#00FF55").pack(anchor="w", padx=30, pady=(6, 2))
        tk.Checkbutton(self.scrollable_frame, text=txt["frameless"], variable=self.frameless_var, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 9, "bold"), activebackground="#111111", activeforeground="#00FF55").pack(anchor="w", padx=30, pady=(2, 4))

        tk.Label(self.scrollable_frame, text=txt["fade_style"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(2, 0))
        tk.Radiobutton(self.scrollable_frame, text=txt["fade_opt1"], variable=self.fade_mode_var, value="fade", fg="#00FF00", bg="#111111", selectcolor="#222222", font=("Consolas", 9)).pack(anchor="w", padx=50)
        tk.Radiobutton(self.scrollable_frame, text=txt["fade_opt2"], variable=self.fade_mode_var, value="out_of_sight", fg="#00FF00", bg="#111111", selectcolor="#222222", font=("Consolas", 9)).pack(anchor="w", padx=50)
        tk.Radiobutton(self.scrollable_frame, text=txt["fade_opt3"], variable=self.fade_mode_var, value="bottom_fade", fg="#00FF00", bg="#111111", selectcolor="#222222", font=("Consolas", 9)).pack(anchor="w", padx=50)

        tk.Label(self.scrollable_frame, text=txt["bg_trans"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(2, 0))
        
        self.bg_color_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        self.bg_color_frame.pack(anchor="w", padx=50, pady=2)

        bg_color_choices = [
            "#000000 - Classic Black", "#050b14 - Dark Navy", "#121212 - Dark Charcoal",
            "#001100 - Dark Matrix Green", "#1a001a - Deep Purple", "#1a0000 - Blood Dark Red",
            "#001a1a - Dark Cyber Cyan", "#FFFFFF - Pure White"
        ]
        self.bg_dropdown = ttk.Combobox(self.bg_color_frame, textvariable=self.bg_combo_var, values=bg_color_choices, width=28, state="readonly")
        self.bg_dropdown.pack(side=tk.LEFT, padx=5)

        bg_btn_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        bg_btn_frame.pack(anchor="w", padx=50, pady=2)
        
        self.btn_bg = tk.Button(bg_btn_frame, text=txt["pick_bg"], command=self.pick_bg, font=("Consolas", 8), bg="#222222", fg="#FFFFFF")
        self.btn_bg.pack(side=tk.LEFT, padx=5)
        self.btn_bg_reset = tk.Button(bg_btn_frame, text=txt["reset"], command=self.reset_bg, font=("Consolas", 8), bg="#331111", fg="#FFFFFF", state="disabled")
        self.btn_bg_reset.pack(side=tk.LEFT)

        tk.Label(self.scrollable_frame, text=txt["pools"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(4, 0))
        
        pools_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        pools_frame.pack(anchor="w", padx=50, pady=2)

        tk.Checkbutton(pools_frame, text=txt["pool_bin"], variable=self.pool_bin, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=0, column=0, sticky="w")
        tk.Checkbutton(pools_frame, text=txt["pool_hex"], variable=self.pool_hex, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=0, column=1, sticky="w", padx=10)
        tk.Checkbutton(pools_frame, text=txt["pool_eng"], variable=self.pool_eng, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=1, column=0, sticky="w")
        tk.Checkbutton(pools_frame, text=txt["pool_arab"], variable=self.pool_arab, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=1, column=1, sticky="w", padx=10)
        tk.Checkbutton(pools_frame, text=txt["pool_kata"], variable=self.pool_kata, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=2, column=0, sticky="w")
        tk.Checkbutton(pools_frame, text=txt["pool_sym"], variable=self.pool_sym, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").grid(row=2, column=1, sticky="w", padx=10)

        tk.Label(self.scrollable_frame, text=txt["custom_chars"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).pack(anchor="w", padx=30, pady=(2, 0))
        self.custom_chars_entry = tk.Entry(self.scrollable_frame, width=44, font=("Consolas", 9))
        self.custom_chars_entry.insert(0, "MATRIX 0123456789")
        self.custom_chars_entry.pack(anchor="w", padx=50, pady=2)

        tk.Label(self.scrollable_frame, text=txt["text_color"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(4, 0))
        
        self.color_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        self.color_frame.pack(anchor="w", padx=50, pady=2)

        color_choices = [
            "#00FF00 - Classic Green", "#00FFFF - Cyber Neon Cyan", "#FF0055 - Cyberpunk Neon Pink",
            "#FF0000 - Blood Red", "#FFFF00 - Yellow", "#9400D3 - Electric Purple",
            "#FF8C00 - Dark Orange", "#00FF66 - Lime Green", "#FFFFFF - Pure White"
        ]
        self.color_dropdown = ttk.Combobox(self.color_frame, textvariable=self.color_combo_var, values=color_choices, width=28, state="readonly")
        self.color_dropdown.pack(side=tk.LEFT, padx=5)

        fg_btn_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        fg_btn_frame.pack(anchor="w", padx=50, pady=2)

        self.btn_fg = tk.Button(fg_btn_frame, text=txt["pick_fg"], command=self.pick_fg, font=("Consolas", 8), bg="#222222", fg="#FFFFFF")
        self.btn_fg.pack(side=tk.LEFT, padx=5)
        self.btn_fg_reset = tk.Button(fg_btn_frame, text=txt["reset"], command=self.reset_fg, font=("Consolas", 8), bg="#331111", fg="#FFFFFF", state="disabled")
        self.btn_fg_reset.pack(side=tk.LEFT)

        tk.Label(self.scrollable_frame, text=txt["adv_settings"], fg="#FFFFFF", bg="#111111", font=("Consolas", 9)).pack(anchor="w", padx=30, pady=(6, 0))
        
        settings_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        settings_frame.pack(anchor="w", padx=50, pady=2)

        tk.Label(settings_frame, text=txt["speed"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=0, column=0, sticky="w")
        self.speed_entry = tk.Entry(settings_frame, width=8, font=("Consolas", 9))
        self.speed_entry.insert(0, "30")
        self.speed_entry.grid(row=0, column=1, padx=5, pady=2)

        tk.Label(settings_frame, text=txt["font_size"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=1, column=0, sticky="w")
        self.fontsize_entry = tk.Entry(settings_frame, width=8, font=("Consolas", 9))
        self.fontsize_entry.insert(0, "14")
        self.fontsize_entry.grid(row=1, column=1, padx=5, pady=2)

        tk.Label(settings_frame, text=txt["tail_len"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=2, column=0, sticky="w")
        self.tail_entry = tk.Entry(settings_frame, width=8, font=("Consolas", 9))
        self.tail_entry.insert(0, "18")
        self.tail_entry.grid(row=2, column=1, padx=5, pady=2)

        tk.Label(settings_frame, text=txt["win_alpha"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=3, column=0, sticky="w")
        alpha_sub_frame = tk.Frame(settings_frame, bg="#111111")
        alpha_sub_frame.grid(row=3, column=1, padx=5, pady=2, sticky="w")

        self.alpha_entry = tk.Entry(alpha_sub_frame, width=5, font=("Consolas", 9))
        self.alpha_entry.insert(0, "0.95")
        self.alpha_entry.pack(side=tk.LEFT)

        btn_less_alpha = tk.Button(alpha_sub_frame, text="-", width=2, command=self.decrease_alpha, bg="#333333", fg="#FFFFFF", font=("Consolas", 8, "bold"))
        btn_less_alpha.pack(side=tk.LEFT, padx=2)
        btn_more_alpha = tk.Button(alpha_sub_frame, text="+", width=2, command=self.increase_alpha, bg="#333333", fg="#FFFFFF", font=("Consolas", 8, "bold"))
        btn_more_alpha.pack(side=tk.LEFT)

        tk.Label(settings_frame, text=txt["bg_alpha_toggle"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=4, column=0, sticky="w")
        self.bg_alpha_btn = tk.Button(
            settings_frame, 
            textvariable=tk.StringVar(value="Normal (1.0)"), 
            command=self.toggle_bg_alpha_btn, 
            bg="#222222", 
            fg="#00FF00", 
            font=("Consolas", 8, "bold"), 
            width=15
        )
        self.bg_alpha_btn.grid(row=4, column=1, padx=5, pady=2, sticky="w")
        self.update_bg_alpha_btn_display()

        tk.Label(settings_frame, text=txt["text_alpha"], fg="#AAAAAA", bg="#111111", font=("Consolas", 8)).grid(row=5, column=0, sticky="w")
        text_alpha_sub_frame = tk.Frame(settings_frame, bg="#111111")
        text_alpha_sub_frame.grid(row=5, column=1, padx=5, pady=2, sticky="w")

        self.text_alpha_entry = tk.Entry(text_alpha_sub_frame, width=5, font=("Consolas", 9))
        self.text_alpha_entry.insert(0, "1.00")
        self.text_alpha_entry.pack(side=tk.LEFT)

        btn_less_text_alpha = tk.Button(text_alpha_sub_frame, text="-", width=2, command=self.decrease_text_alpha, bg="#333333", fg="#FFFFFF", font=("Consolas", 8, "bold"))
        btn_less_text_alpha.pack(side=tk.LEFT, padx=2)
        btn_more_text_alpha = tk.Button(text_alpha_sub_frame, text="+", width=2, command=self.increase_text_alpha, bg="#333333", fg="#FFFFFF", font=("Consolas", 8, "bold"))
        btn_more_text_alpha.pack(side=tk.LEFT)

        tk.Checkbutton(self.scrollable_frame, text=txt["white_head"], variable=self.white_head_var, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").pack(anchor="w", padx=50, pady=(4, 1))

        head_color_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        head_color_frame.pack(anchor="w", padx=50, pady=2)
        self.btn_head_color = tk.Button(head_color_frame, text=txt["pick_head"], command=self.pick_head_color, font=("Consolas", 8), bg="#FFFFFF", fg="#000000", width=32)
        self.btn_head_color.pack(side=tk.LEFT, padx=5)
        self.btn_head_reset = tk.Button(head_color_frame, text=txt["reset_head"], command=self.reset_head_color, font=("Consolas", 8), bg="#333333", fg="#FFFFFF")
        self.btn_head_reset.pack(side=tk.LEFT)

        tk.Checkbutton(self.scrollable_frame, text=txt["tail_tip"], variable=self.custom_tail_tip_var, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").pack(anchor="w", padx=50, pady=1)

        tk.Checkbutton(self.scrollable_frame, text=txt["dynamic_shuffle"], variable=self.dynamic_shuffle_var, fg="#00FF55", bg="#111111", selectcolor="#222222", font=("Consolas", 8), activebackground="#111111", activeforeground="#00FF55").pack(anchor="w", padx=50, pady=1)

        btn_action_frame = tk.Frame(self.scrollable_frame, bg="#111111")
        btn_action_frame.pack(pady=20)

        btn_start = tk.Button(btn_action_frame, text=txt["launch"], command=self.start_matrix, bg="#00FF00", fg="#000000", font=("Consolas", 10, "bold"), width=15)
        btn_start.pack(side=tk.LEFT, padx=5)

        btn_save = tk.Button(btn_action_frame, text=txt["save"], command=self.save_config_to_file, bg="#0088FF", fg="#FFFFFF", font=("Consolas", 10, "bold"), width=13)
        btn_save.pack(side=tk.LEFT, padx=5)

    def toggle_bg_alpha_btn(self):
        current = self.bg_alpha_state.get()
        self.bg_alpha_state.set(not current)
        self.update_bg_alpha_btn_display()

    def update_bg_alpha_btn_display(self):
        if self.bg_alpha_state.get():
            self.bg_alpha_btn.config(text="Transparent (0.0)", bg="#331111", fg="#FF5555")
        else:
            self.bg_alpha_btn.config(text="Normal (1.0)", bg="#222222", fg="#00FF00")

    def decrease_alpha(self):
        try:
            val = float(self.alpha_entry.get())
            val = max(0.1, val - 0.05)
            self.alpha_entry.delete(0, tk.END)
            self.alpha_entry.insert(0, f"{val:.2f}")
        except ValueError:
            self.alpha_entry.delete(0, tk.END)
            self.alpha_entry.insert(0, "0.90")

    def increase_alpha(self):
        try:
            val = float(self.alpha_entry.get())
            val = min(1.0, val + 0.05)
            self.alpha_entry.delete(0, tk.END)
            self.alpha_entry.insert(0, f"{val:.2f}")
        except ValueError:
            self.alpha_entry.delete(0, tk.END)
            self.alpha_entry.insert(0, "0.95")

    def decrease_text_alpha(self):
        try:
            val = float(self.text_alpha_entry.get())
            val = max(0.1, val - 0.05)
            self.text_alpha_entry.delete(0, tk.END)
            self.text_alpha_entry.insert(0, f"{val:.2f}")
        except ValueError:
            self.text_alpha_entry.delete(0, tk.END)
            self.text_alpha_entry.insert(0, "0.95")

    def increase_text_alpha(self):
        try:
            val = float(self.text_alpha_entry.get())
            val = min(1.0, val + 0.05)
            self.text_alpha_entry.delete(0, tk.END)
            self.text_alpha_entry.insert(0, f"{val:.2f}")
        except ValueError:
            self.text_alpha_entry.delete(0, tk.END)
            self.text_alpha_entry.insert(0, "1.00")

    def pick_bg(self):
        c = colorchooser.askcolor(title="Choose Custom Background")[1]
        if c:
            self.custom_bg = c
            self.bg_dropdown.pack_forget() 
            self.btn_bg.config(text=f"Custom BG: {c}", bg=c, fg="#000000" if sum(hex_to_rgb(c)) > 380 else "#FFFFFF")
            self.btn_bg_reset.config(state="normal")

    def reset_bg(self):
        self.custom_bg = None
        self.bg_dropdown.pack(side=tk.LEFT, padx=5) 
        self.btn_bg.config(text=LANG[self.current_lang]["pick_bg"], bg="#222222", fg="#FFFFFF")
        self.btn_bg_reset.config(state="disabled")

    def pick_fg(self):
        c = colorchooser.askcolor(title="Choose Custom Text Color")[1]
        if c:
            self.custom_fg = c
            self.color_dropdown.pack_forget() 
            self.btn_fg.config(text=f"Custom Text: {c}", bg=c, fg="#000000" if sum(hex_to_rgb(c)) > 380 else "#FFFFFF")
            self.btn_fg_reset.config(state="normal")

    def reset_fg(self):
        self.custom_fg = None
        self.color_dropdown.pack(side=tk.LEFT, padx=5) 
        self.btn_fg.config(text=LANG[self.current_lang]["pick_fg"], bg="#222222", fg="#FFFFFF")
        self.btn_fg_reset.config(state="disabled")

    def pick_head_color(self):
        c = colorchooser.askcolor(title="Choose Leading Character Color")[1]
        if c:
            self.custom_head_color = c
            self.btn_head_color.config(text=f"Leading Color: {c}", bg=c, fg="#000000" if sum(hex_to_rgb(c)) > 380 else "#FFFFFF")

    def reset_head_color(self):
        self.custom_head_color = None
        self.btn_head_color.config(text=LANG[self.current_lang]["pick_head"], bg="#FFFFFF", fg="#000000")

    def get_config_dict(self):
        try:
            color = self.custom_fg if self.custom_fg else self.color_combo_var.get().split(" - ")[0]
            bg_color = self.custom_bg if self.custom_bg else self.bg_combo_var.get().split(" - ")[0]
            head_color = self.custom_head_color if self.custom_head_color else "#FFFFFF"

            speed = int(self.speed_entry.get())
            font_size = int(self.fontsize_entry.get())
            tail_length = int(self.tail_entry.get())
            alpha = float(self.alpha_entry.get())
            bg_alpha = 0.0 if self.bg_alpha_state.get() else 1.0
            text_alpha = float(self.text_alpha_entry.get())
            terminal_title = self.terminal_title_entry.get().strip() or "Matrix Engine"
        except ValueError:
            color, bg_color, head_color = "#00FF00", "#000000", "#FFFFFF"
            speed, font_size, tail_length, alpha, bg_alpha, text_alpha = 30, 14, 18, 0.95, 1.0, 1.0
            terminal_title = "Matrix Engine"

        return {
            "lang": self.current_lang,
            "terminal_title": terminal_title,
            "use_bin": self.pool_bin.get(),
            "use_hex": self.pool_hex.get(),
            "use_eng": self.pool_eng.get(),
            "use_arab": self.pool_arab.get(),
            "use_kata": self.pool_kata.get(),
            "use_sym": self.pool_sym.get(),
            "custom_text": self.custom_chars_entry.get(),
            "color": color,
            "custom_fg": self.custom_fg,
            "color_combo": self.color_combo_var.get(),
            "head_color": head_color,
            "custom_head_color": self.custom_head_color,
            "is_fullscreen": self.fullscreen_var.get(),
            "always_on_top": self.always_on_top_var.get(),
            "frameless": self.frameless_var.get(),
            "fade_mode": self.fade_mode_var.get(),
            "bg_color": bg_color,
            "custom_bg": self.custom_bg,
            "bg_combo": self.bg_combo_var.get(),
            "speed": speed,
            "font_size": font_size,
            "tail_length": tail_length,
            "alpha": alpha,
            "bg_alpha": bg_alpha,
            "text_alpha": text_alpha,
            "white_head": self.white_head_var.get(),
            "custom_tail_tip": self.custom_tail_tip_var.get(),
            "dynamic_shuffle": self.dynamic_shuffle_var.get()
        }

    def save_config_to_file(self):
        cfg = self.get_config_dict()
        try:
            with open(CONFIG_FILE, "w", encoding="utf-8") as f:
                json.dump(cfg, f, indent=4)
            messagebox.showinfo("Success", LANG[self.current_lang]["success_save"])
        except Exception as e:
            messagebox.showerror("Error", f"{LANG[self.current_lang]['err_save']}\n{e}")

    def load_config_from_file(self):
        if not os.path.exists(CONFIG_FILE):
            return
        try:
            with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                cfg = json.load(f)

            self.terminal_title_entry.delete(0, tk.END)
            self.terminal_title_entry.insert(0, cfg.get("terminal_title", "Matrix Engine"))

            self.fullscreen_var.set(cfg.get("is_fullscreen", False))
            self.always_on_top_var.set(cfg.get("always_on_top", False))
            self.frameless_var.set(cfg.get("frameless", False))
            self.fade_mode_var.set(cfg.get("fade_mode", "out_of_sight"))

            self.custom_bg = cfg.get("custom_bg")
            if self.custom_bg:
                self.bg_dropdown.pack_forget()
                self.btn_bg.config(text=f"Custom BG: {self.custom_bg}", bg=self.custom_bg, fg="#000000" if sum(hex_to_rgb(self.custom_bg)) > 380 else "#FFFFFF")
                self.btn_bg_reset.config(state="normal")
            else:
                self.bg_combo_var.set(cfg.get("bg_combo", "#000000 - Classic Black"))

            self.pool_bin.set(cfg.get("use_bin", True))
            self.pool_hex.set(cfg.get("use_hex", True))
            self.pool_eng.set(cfg.get("use_eng", True))
            self.pool_arab.set(cfg.get("use_arab", False))
            self.pool_kata.set(cfg.get("use_kata", True))
            self.pool_sym.set(cfg.get("use_sym", True))

            self.custom_chars_entry.delete(0, tk.END)
            self.custom_chars_entry.insert(0, cfg.get("custom_text", ""))

            self.custom_fg = cfg.get("custom_fg")
            if self.custom_fg:
                self.color_dropdown.pack_forget()
                self.btn_fg.config(text=f"Custom Text: {self.custom_fg}", bg=self.custom_fg, fg="#000000" if sum(hex_to_rgb(self.custom_fg)) > 380 else "#FFFFFF")
                self.btn_fg_reset.config(state="normal")
            else:
                self.color_combo_var.set(cfg.get("color_combo", "#00FF00 - Classic Green"))

            self.custom_head_color = cfg.get("custom_head_color")
            if self.custom_head_color:
                self.btn_head_color.config(text=f"Leading Color: {self.custom_head_color}", bg=self.custom_head_color, fg="#000000" if sum(hex_to_rgb(self.custom_head_color)) > 380 else "#FFFFFF")

            self.speed_entry.delete(0, tk.END)
            self.speed_entry.insert(0, str(cfg.get("speed", 30)))

            self.fontsize_entry.delete(0, tk.END)
            self.fontsize_entry.insert(0, str(cfg.get("font_size", 14)))

            self.tail_entry.delete(0, tk.END)
            self.tail_entry.insert(0, str(cfg.get("tail_length", 18)))

            self.alpha_entry.delete(0, tk.END)
            self.alpha_entry.insert(0, str(cfg.get("alpha", 0.95)))

            saved_bg_alpha = cfg.get("bg_alpha", 1.0)
            self.bg_alpha_state.set(saved_bg_alpha == 0.0)
            self.update_bg_alpha_btn_display()

            self.text_alpha_entry.delete(0, tk.END)
            self.text_alpha_entry.insert(0, str(cfg.get("text_alpha", 1.0)))

            self.white_head_var.set(cfg.get("white_head", True))
            self.custom_tail_tip_var.set(cfg.get("custom_tail_tip", False))
            self.dynamic_shuffle_var.set(cfg.get("dynamic_shuffle", True))

        except Exception:
            pass

    def start_matrix(self):
        config_data = self.get_config_dict()
        self.root.destroy()
        run_matrix_window(config_data)

def return_to_launcher(matrix_root, lang="ar"):
    matrix_root.destroy()
    new_root = tk.Tk()
    MatrixLauncher(new_root, selected_lang=lang)
    new_root.mainloop()

def run_matrix_window(config):
    root = tk.Tk()
    root.title(config["terminal_title"])
    lang = config.get("lang", "ar")
    
    if config.get("frameless", False):
        root.overrideredirect(True)
    
    bg_color_str = config["bg_color"]
    bg_rgb = hex_to_rgb(bg_color_str)
    bg_alpha = config.get("bg_alpha", 1.0)
    
    try:
        if bg_alpha == 0.0:
            try:
                root.wm_attributes("-transparentcolor", bg_color_str)
            except tk.TclError:
                pass
            blended_bg = bg_color_str
        else:
            final_bg_r = int(bg_rgb[0] * bg_alpha + 17 * (1 - bg_alpha))
            final_bg_g = int(bg_rgb[1] * bg_alpha + 17 * (1 - bg_alpha))
            final_bg_b = int(bg_rgb[2] * bg_alpha + 17 * (1 - bg_alpha))
            blended_bg = rgb_to_hex((final_bg_r, final_bg_g, final_bg_b))
            
        root.attributes("-alpha", config["alpha"])
        
    except tk.TclError:
        if sys.platform.startswith("linux"):
            proceed = messagebox.askyesno(
                LANG[lang]["warn_title"],
                "It seems your Linux environment does not support window transparency (missing Compositor).\nDo you want to open the window without transparency?"
            )
            if not proceed:
                root.destroy()
                return
        blended_bg = bg_color_str

    root.configure(bg=blended_bg)

    if config["is_fullscreen"]:
        screen_width = root.winfo_screenwidth()
        screen_height = root.winfo_screenheight()
        root.geometry(f"{screen_width}x{screen_height}+0+0")
        root.attributes("-fullscreen", True)
    else:
        screen_width, screen_height = 1000, 700
        root.geometry(f"{screen_width}x{screen_height}")

    if config.get("always_on_top", False):
        root.attributes("-topmost", True)

    def start_move(event):
        root._x = event.x
        root._y = event.y

    def do_move(event):
        x = event.x_root - root._x
        y = event.y_root - root._y
        root.geometry(f"+{x}+{y}")

    if config.get("frameless", False):
        root.bind("<Button-1>", start_move)
        root.bind("<B1-Motion>", do_move)

    btn_back = tk.Button(
        root, text=LANG[lang]["back_btn"], 
        command=lambda: return_to_launcher(root, lang=lang),
        bg="#222222", fg="#00FF00", font=("Consolas", 9, "bold"),
        activebackground="#00FF00", activeforeground="#000000",
        relief="flat", bd=2
    )
    btn_back.place(x=15, y=15)

    font_name = "Consolas"
    canvas = tk.Canvas(root, bg=blended_bg, highlightthickness=0)
    canvas.pack(fill=tk.BOTH, expand=True)

    char_pool = []
    if config["use_bin"]: char_pool.extend(['0', '1'])
    if config["use_hex"]: char_pool.extend([str(i) for i in range(10)] + ['A', 'B', 'C', 'D', 'E', 'F'])
    if config["use_eng"]: char_pool.extend([chr(i) for i in range(65, 91)] + [chr(i) for i in range(97, 123)])
    if config["use_arab"]: char_pool.extend([chr(i) for i in range(0x0621, 0x064A)])
    if config["use_kata"]: char_pool.extend([chr(i) for i in range(0x30A0, 0x30FF)])
    if config["use_sym"]: char_pool.extend(['!', '@', '#', '$', '%', '^', '&', '*', '(', ')', '_', '+', '-', '=', '[', ']', '{', '}', ';', ':', ',', '.', '<', '>', '?', '/'])
    if config["custom_text"]: char_pool.extend(list(config["custom_text"]))

    if not char_pool:
        char_pool = ['0', '1', 'A', 'B']

    col_width = config["font_size"] + 2
    row_height = config["font_size"] + 4
    total_cols = screen_width // col_width
    rows = screen_height // row_height
    base_len = config["tail_length"]
    base_rgb = hex_to_rgb(config["color"])
    head_rgb = hex_to_rgb(config["head_color"])
    text_alpha = config.get("text_alpha", 1.0)

    num_active_columns = int(total_cols * 0.85)
    
    columns_data = []
    for _ in range(num_active_columns):
        col_x_index = random.randint(0, total_cols - 1)
        columns_data.append({
            "x_idx": col_x_index,
            "drop": random.randint(-rows, rows),
            "speed": random.choice([1, 1, 2]),
            "length": random.randint(max(8, int(base_len * 0.7)), int(base_len * 1.3)),
            "delay": random.randint(0, 15),
            "chars": [random.choice(char_pool) for _ in range(rows * 3)]
        })

    is_running = True

    def update_matrix():
        nonlocal is_running
        if not is_running:
            return

        canvas.delete("matrix")
        
        for c_data in columns_data:
            if c_data["delay"] > 0:
                c_data["delay"] -= 1
                continue

            x = c_data["x_idx"] * col_width
            head_row = c_data["drop"]
            t_len = c_data["length"]
            chars_list = c_data["chars"]

            for i in range(t_len):
                current_row = head_row - i
                char_y = current_row * row_height
                if -row_height <= char_y <= screen_height + row_height:
                    row_idx = current_row % len(chars_list)
                    if config["dynamic_shuffle"] and random.random() < 0.35:
                        chars_list[row_idx] = random.choice(char_pool)
                    
                    char = chars_list[row_idx] if 0 <= row_idx < len(chars_list) else random.choice(char_pool)

                    fade_factor = 1.0
                    if i == 0 and config["white_head"]:
                        r, g, b = head_rgb[0], head_rgb[1], head_rgb[2]
                    elif i == t_len - 1 and config["custom_tail_tip"]:
                        r, g, b = 119, 119, 119
                    else:
                        if config["fade_mode"] == "fade":
                            fade_factor = max(0.0, 1.0 - (i / t_len))
                        elif config["fade_mode"] == "bottom_fade":
                            distance_from_bottom = screen_height - char_y
                            fade_zone = screen_height * 0.35 
                            if distance_from_bottom < fade_zone:
                                fade_factor = max(0.0, distance_from_bottom / fade_zone)
                            fade_factor *= max(0.0, 1.0 - (i / t_len))
                        else:
                            fade_factor = 1.0

                        r = int(base_rgb[0] * fade_factor)
                        g = int(base_rgb[1] * fade_factor)
                        b = int(base_rgb[2] * fade_factor)
                        
                    final_r = int(r * text_alpha + bg_rgb[0] * (1 - text_alpha))
                    final_g = int(g * text_alpha + bg_rgb[1] * (1 - text_alpha))
                    final_b = int(b * text_alpha + bg_rgb[2] * (1 - text_alpha))
                    color = rgb_to_hex((final_r, final_g, final_b))
                        
                    canvas.create_text(x, char_y, text=char, fill=color, font=(font_name, config["font_size"], "bold"), tag="matrix")

            c_data["drop"] += c_data["speed"]
            
            if (c_data["drop"] - t_len) * row_height > screen_height:
                c_data["x_idx"] = random.randint(0, total_cols - 1)
                c_data["drop"] = random.randint(-int(rows * 1.2), -2)
                c_data["speed"] = random.choice([1, 1, 2])
                c_data["length"] = random.randint(max(8, int(base_len * 0.7)), int(base_len * 1.3))
                c_data["delay"] = random.randint(0, 15)
                c_data["chars"] = [random.choice(char_pool) for _ in range(rows * 3)]
                
        root.after(config["speed"], update_matrix)

    def close_window(e=None):
        nonlocal is_running
        is_running = False
        try:
            root.destroy()
        except Exception:
            pass
        return_to_launcher(root, lang=lang)

    root.bind("<Escape>", close_window)
    root.bind_all("~", close_window)
    root.bind_all("`", close_window)
    root.bind_all("¬", close_window) 

    hook_id = None
    if sys.platform.startswith("win"):
        try:
            PHANDLER = ctypes.WINFUNCTYPE(ctypes.c_int, ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_void_p))
            
            def global_keyboard_callback(nCode, wParam, lParam):
                if nCode >= 0 and wParam in (0x0100, 0x0104):
                    vk_code = ctypes.c_int.from_address(lParam).value
                    if vk_code == 0xC0 or vk_code == 223: 
                        root.after(0, close_window)
                user32 = ctypes.windll.user32
                return user32.CallNextHookEx(hook_id, nCode, wParam, lParam)

            callback_func = PHANDLER(global_keyboard_callback)
            HINSTANCE = ctypes.windll.kernel32.GetModuleHandleW(None)
            hook_id = ctypes.windll.user32.SetWindowsHookExW(13, callback_func, HINSTANCE, 0)
        except Exception:
            pass

    update_matrix()
    root.mainloop()

    if sys.platform.startswith("win") and hook_id:
        try:
            ctypes.windll.user32.UnhookWindowsHookEx(hook_id)
        except Exception:
            pass

if __name__ == "__main__":
    saved_lang = "ar"
    if os.path.exists(CONFIG_FILE):
        try:
            with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                saved_cfg = json.load(f)
                saved_lang = saved_cfg.get("lang", "ar")
        except Exception:
            pass

    lang_root = tk.Tk()
    lang_selector = LanguageSelector(lang_root)
    lang_selector.selected_lang = saved_lang
    lang_root.mainloop()

    setup_root = tk.Tk()
    MatrixLauncher(setup_root, selected_lang=lang_selector.selected_lang)
    setup_root.mainloop()
