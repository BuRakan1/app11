class ModernTrainingDesign(tk.Tk):
    """تطبيق إدارة التدريب - قالب فارغ"""

    APP_NAME = "نظام إدارة التدريب"
    BASE_PATH = getattr(sys, "_MEIPASS", os.path.dirname(os.path.abspath(__file__)))
    FONT_PATH = os.path.join(BASE_PATH, "assets", "fonts", "Tajawal-Regular.ttf")

    # نظام الألوان
    COLORS = {
        "primary": "#1a73e8",
        "secondary": "#4285f4",
        "success": "#34a853",
        "danger": "#ea4335",
        "warning": "#fbbc05",
        "light": "#f5f5f0",  # بيج فاتح بدلاً من الأبيض
        "dark": "#202124",
        "surface": "#f8f8f3",  # رمادي دافئ فاتح جداً بدلاً من الأبيض
        "background": "#e8e8e3",  # رمادي دافئ للخلفية
        "card": "#f5f5f0",  # بيج فاتح للكروت
        "on_surface": "#202124",
        "border": "#d5d5d0",  # حدود رمادية دافئة
        "table_bg": "#f0f0eb",  # خلفية الجداول
        "table_alt": "#e5e5e0",  # صفوف متناوبة
        "input_bg": "#fafaf5",  # خلفية حقول الإدخال
    }

    # — حدّث/أضف داخل القاموس FONTS —
    FONTS = {
        "large_title": ("Tajawal", 24, "bold"),
        "title": ("Tajawal", 18, "bold"),
        "subtitle": ("Tajawal", 16, "bold"),
        "text": ("Tajawal", 12),
        "text_bold": ("Tajawal", 12, "bold"),
        "small": ("Tajawal", 10),
        "nav": ("Tajawal", 20, "bold"),  # ← خذ مقاس أكبر للزرار العلوية
        "body": ("Tajawal", 14),
        "large": ("Tajawal", 16, "bold"),
    }

    def __init__(self, root=None, current_user=None, db_conn=None):
        super().__init__()

        self.title("نظام إدارة التدريب - قسم تصميم وتطوير البرامج")
        self.current_user = current_user
        self.db_conn = db_conn

        # التكيف مع حجم الشاشة تلقائياً
        screen_width = self.winfo_screenwidth()
        screen_height = self.winfo_screenheight()

        app_width = min(1400, int(screen_width * 0.90))
        app_height = min(800, int(screen_height * 0.90))

        x = (screen_width - app_width) // 2
        y = (screen_height - app_height) // 2

        self.geometry(f"{app_width}x{app_height}+{x}+{y}")
        self.minsize(800, 600)
        self.configure(bg=self.COLORS["background"])

        self.screen_width = screen_width
        self.screen_height = screen_height
        self.app_width = app_width
        self.app_height = app_height

        self._load_font()
        self._setup_styles()

        self.current_page = "الرئيسية"
        self._is_closing = False  # متغير لتتبع حالة الإغلاق

        # إنشاء الجداول إذا لم تكن موجودة
        self._create_database_tables()

        # إنشاء عناصر الواجهة
        self._create_header()
        self._create_tab_control()
        self._create_footer()

        # معالجة إغلاق النافذة بشكل صحيح
        self.protocol("WM_DELETE_WINDOW", self._on_closing)

    def _on_closing(self):
        """معالجة إغلاق النافذة بشكل نظيف"""
        if self._is_closing:
            return

        self._is_closing = True

        try:
            # إلغاء جميع الأحداث المجدولة
            for widget in self.winfo_children():
                try:
                    widget.destroy()
                except:
                    pass

            # إغلاق اتصال قاعدة البيانات
            if hasattr(self, 'db_conn') and self.db_conn:
                try:
                    self.db_conn.close()
                except:
                    pass

            # إيقاف mainloop
            self.quit()

        except:
            pass
        finally:
            try:
                self.destroy()
            except:
                pass

            # إنهاء البرنامج
            import sys
            sys.exit(0)

    def _load_font(self):
        """تحميل خط Tajawal"""
        if os.path.isfile(self.FONT_PATH):
            try:
                tkfont.nametofont("Tajawal")
            except tk.TclError:
                self.tk.call("font", "create", "Tajawal", "-family", "Tajawal", "-size", 12)
                self.tk.call("font", "configure", "Tajawal", "-file", self.FONT_PATH)

    def _setup_styles(self):
        """تبويبات رمادية داكنة، تكبير بسيط، خط أسود"""
        try:
            style = ttk.Style(self)

            # التحقق من توفر الثيم
            available_themes = style.theme_names()
            if "clam" in available_themes:
                style.theme_use("clam")
            else:
                # استخدام الثيم الافتراضي إذا لم يكن clam متوفراً
                style.theme_use("default")

            # ألوان مطلوبة
            base_gray = "#8c8c8c"  # رمادي أغمق للزر غير المحدَّد
            hover_gray = "#a8a8a8"  # رمادي متوسط عند المرور
            selected_color = "#3B82F6"  # أزرق للتبويبة المحددة
            txt_color = "black"
            selected_txt = "black"  # نص أسود للتبويبة المحددة
            notebook_bg = "#e8e8e8"  # رمادي فاتح للخلفية

            # Notebook بخلفية رمادية فاتحة
            style.configure("Bold.TNotebook",
                            background=notebook_bg,  # تغيير الخلفية إلى رمادي فاتح
                            borderwidth=0,
                            relief="flat",
                            tabmargins=[2, 5, 2, 0])  # إضافة مسافات بين التبويبات

            # تبويبات أكبر قليلاً: خط أكبر وبادينغ أكثر
            style.configure("Bold.TNotebook.Tab",
                            font=self.FONTS["subtitle"],  # ← حجم 16 عريض
                            padding=[16, 8],  # ← عرض وارتفاع أكبر
                            borderwidth=0,
                            relief="flat",
                            background=base_gray,
                            foreground=txt_color)

            # تغيّر اللون عند الاختيار أو المرور
            style.map("Bold.TNotebook.Tab",
                      background=[("selected", selected_color),
                                  ("active", hover_gray)],
                      foreground=[("selected", selected_txt),
                                  ("active", txt_color)])

            # إزالة خط التركيز نهائيًا
            try:
                style.layout("Bold.TNotebook.Tab",
                             [('Notebook.padding', {'side': 'top', 'sticky': 'nswe', 'children': [
                                 ('Notebook.label', {'side': 'top', 'sticky': ''})
                             ]})])
            except:
                # إذا فشل تخصيص التخطيط، نتجاهل الخطأ
                pass

        except Exception as e:
            # في حالة حدوث أي خطأ، نستمر بدون تخصيص الأنماط
            print(f"خطأ في إعداد الأنماط: {e}")

    def _create_header(self):
        """إلغاء الشريط الأزرق العلوي نهائيًّا."""
        return  # ما نرسم أي إطار أو أزرار

    def _cleanup_window(self, window):
        """تنظيف الذاكرة عند إغلاق النافذة"""
        try:
            # إلغاء جميع الأحداث المجدولة
            for after_id in window.tk.call('after', 'info'):
                window.after_cancel(after_id)

            # تدمير جميع العناصر الفرعية
            for widget in window.winfo_children():
                widget.destroy()

            # تدمير النافذة
            window.destroy()

            # استدعاء جامع القمامة
            import gc
            gc.collect()

        except:
            pass

    def _create_tab_control(self):
        """إنشاء التبويبات"""
        # إطار التبويبات
        tab_frame = tk.Frame(self, bg=self.COLORS["background"])
        tab_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        self.tab_control = ttk.Notebook(tab_frame, style="Bold.TNotebook")
        self.tab_control.pack(fill=tk.BOTH, expand=True)

        # إنشاء التبويبات
        self._create_home_tab()
        self._create_teachers_tab()
        self._create_courses_tab()
        if self.current_user and self.current_user["permissions"]["is_admin"]:
            self._create_users_tab()
        self._create_settings_tab()

    def _create_footer(self):
        """شريط سفلي محسّن مع معلومات النظام وزر الخروج"""
        footer = tk.Frame(self, bg="#2c3e50", height=50)
        footer.pack(side=tk.BOTTOM, fill=tk.X)
        footer.pack_propagate(False)

        # الجانب الأيسر - معلومات المستخدم والنظام
        left_frame = tk.Frame(footer, bg="#2c3e50")
        left_frame.pack(side=tk.LEFT, padx=20, pady=8)

        # أيقونة المستخدم
        user_icon = tk.Label(
            left_frame,
            text="👤",
            font=("Arial", 16),
            bg="#2c3e50",
            fg="white"
        )
        user_icon.pack(side=tk.LEFT, padx=(0, 10))

        # معلومات المستخدم
        user_name = tk.Label(
            left_frame,
            text=f"المستخدم: {self.current_user['full_name']}" if self.current_user else "غير معرف",
            font=self.FONTS["text_bold"],
            bg="#2c3e50",
            fg="white"
        )
        user_name.pack(side=tk.LEFT)

        # الوسط - حالة النظام
        center_frame = tk.Frame(footer, bg="#2c3e50")
        center_frame.pack(side=tk.LEFT, expand=True, padx=20)

        status_text = tk.Label(
            center_frame,
            text="إدارة تصميم و تطوير البرامج",
            font=self.FONTS["title"],  # خط أكبر
            bg="#2c3e50",
            fg="white"  # لون أبيض
        )
        status_text.pack(side=tk.LEFT)

        # الجانب الأيمن - زر الخروج
        right_frame = tk.Frame(footer, bg="#2c3e50")
        right_frame.pack(side=tk.RIGHT, padx=20, pady=8)

        # زر تسجيل الخروج
        logout_btn = tk.Button(
            right_frame,
            text="⚡ تسجيل الخروج",
            font=self.FONTS["text_bold"],
            bg="#e74c3c",
            fg="white",
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            padx=20,
            pady=8,
            command=self._logout
        )
        logout_btn.pack(side=tk.RIGHT)

        # تأثير عند المرور على زر الخروج
        def on_enter(e):
            logout_btn['bg'] = '#c0392b'

        def on_leave(e):
            logout_btn['bg'] = '#e74c3c'

        logout_btn.bind("<Enter>", on_enter)
        logout_btn.bind("<Leave>", on_leave)

    def _create_home_tab(self):
        """إنشاء تبويب الصفحة الرئيسية - فارغ"""
        home_frame = tk.Frame(self.tab_control, bg=self.COLORS["background"])
        self.tab_control.add(home_frame, text="الرئيسية")

        # محتوى فارغ
        tk.Label(
            home_frame,
            text="الصفحة الرئيسية",
            font=self.FONTS["large_title"],
            bg=self.COLORS["background"],
            fg=self.COLORS["dark"]
        ).pack(pady=50)

    #— ضفها داخل ModernTrainingDesign —
    def _logout(self):
        """تأكيد الخروج ثم إغلاق النافذة الرئيسية."""
        if messagebox.askyesno("تأكيد", "هل تريد تسجيل الخروج؟"):
            self._on_closing()

    def _create_teachers_tab(self):
        """إنشاء تبويب إدارة المدرسين بتصميم محسن"""
        teachers_frame = tk.Frame(self.tab_control, bg=self.COLORS["background"])
        self.tab_control.add(teachers_frame, text="إدارة المدرسين")

        # إطار العنوان والأزرار
        header_frame = tk.Frame(teachers_frame, bg="#1E3A5F", height=80)  # أزرق داكن رسمي
        header_frame.pack(fill=tk.X, padx=10, pady=10)
        header_frame.pack_propagate(False)

        # عنوان الصفحة
        title_label = tk.Label(
            header_frame,
            text="إدارة المدرسين",
            font=self.FONTS["large_title"],
            bg="#1E3A5F",  # نفس لون الخلفية
            fg="white"  # نص أبيض
        )
        title_label.pack(side=tk.LEFT, padx=20, pady=20)

        # إطار الأزرار
        buttons_frame = tk.Frame(header_frame, bg="#1E3A5F")  # نفس اللون الرسمي
        buttons_frame.pack(side=tk.RIGHT, padx=20, pady=20)

        # زر إضافة مدرس
        add_teacher_btn = tk.Button(
            buttons_frame,
            text="إضافة مدرس جديد",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["success"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._open_add_teacher_window
        )
        add_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر تعديل مدرس
        edit_teacher_btn = tk.Button(
            buttons_frame,
            text="تعديل بيانات مدرس",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["warning"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._edit_teacher
        )
        edit_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر حذف مدرس
        delete_teacher_btn = tk.Button(
            buttons_frame,
            text="حذف مدرس",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["danger"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._delete_teacher
        )
        delete_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر استيراد مدرسين
        import_teacher_btn = tk.Button(
            buttons_frame,
            text="استيراد مدرسين",
            font=self.FONTS["text_bold"],
            bg="#9C27B0",  # لون بنفسجي
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._import_teachers
        )
        import_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر مسارات مدرسي الدورات - محدث
        paths_btn = tk.Button(
            buttons_frame,
            text="📚 مسارات مدرسي الدورات",
            font=self.FONTS["text_bold"],
            bg="#17a2b8",  # لون أزرق فاتح
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._manage_course_teachers_paths  # الدالة الجديدة
        )
        paths_btn.pack(side=tk.LEFT, padx=5)

        # إطار البحث والتصفية
        search_filter_frame = tk.Frame(teachers_frame, bg=self.COLORS["surface"], height=70)
        search_filter_frame.pack(fill=tk.X, padx=15, pady=(10, 5))
        search_filter_frame.pack_propagate(False)

        # إطار داخلي للمحتوى مع توزيع ثابت
        inner_frame = tk.Frame(search_filter_frame, bg=self.COLORS["surface"])
        inner_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        # تكوين الأعمدة بأوزان ثابتة
        inner_frame.grid_columnconfigure(0, weight=1)  # البحث
        inner_frame.grid_columnconfigure(1, weight=1)  # الفلتر
        inner_frame.grid_columnconfigure(2, weight=0)  # زر التصدير

        # 1. إطار البحث (ثابت في اليسار)
        search_container = tk.Frame(inner_frame, bg=self.COLORS["surface"])
        search_container.grid(row=0, column=0, sticky="w", padx=(0, 20))

        tk.Label(
            search_container,
            text="بحث:",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        self.search_entry = tk.Entry(
            search_container,
            font=self.FONTS["text"],
            width=25
        )
        self.search_entry.pack(side=tk.LEFT)

        # ربط حدث الكتابة مباشرة
        def on_search_key(event):
            # تأخير بسيط لتحسين الأداء
            self.after(100, self._apply_filters)

        self.search_entry.bind('<KeyRelease>', on_search_key)

        # 2. إطار التصفية (في الوسط)
        filter_container = tk.Frame(inner_frame, bg=self.COLORS["surface"])
        filter_container.grid(row=0, column=1, sticky="w")

        tk.Label(
            filter_container,
            text="الفئة:",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        # في _create_teachers_tab، تأكد من وجود:
        self.filter_combo = ttk.Combobox(
            filter_container,
            values=["الكل", "منسوبي المدينة", "متعاونين"],
            font=self.FONTS["text"],
            width=20,
            state="readonly"
        )
        self.filter_combo.pack(side=tk.LEFT)
        self.filter_combo.set("الكل")
        self.filter_combo.bind('<<ComboboxSelected>>', lambda e: self._apply_filters())

        # 3. زر التصدير (ثابت في اليمين)
        self.export_btn = tk.Button(
            inner_frame,
            text="📊 تصدير البيانات",
            font=self.FONTS["text_bold"],
            bg="#4CAF50",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._export_filtered_data
        )
        self.export_btn.grid(row=0, column=2, sticky="e")

        # خط فاصل
        separator = tk.Frame(teachers_frame, bg=self.COLORS["border"], height=2)
        separator.pack(fill=tk.X, padx=15, pady=(0, 10))

        # إطار الجدول الرئيسي
        main_table_frame = tk.Frame(teachers_frame, bg=self.COLORS["table_bg"], bd=2, relief=tk.RIDGE)
        main_table_frame.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 10))

        # إطار داخلي للجدول
        table_frame = tk.Frame(main_table_frame, bg=self.COLORS["table_bg"])
        table_frame.pack(fill=tk.BOTH, expand=True, padx=3, pady=3)

        # إنشاء Treeview بتصميم رسمي
        style = ttk.Style()

        # تكوين نمط الجدول
        style.configure("Formal.Treeview",
                        background=self.COLORS["table_bg"],
                        foreground="#2c2c2c",  # نص رمادي داكن بدلاً من الأسود
                        rowheight=45,
                        fieldbackground=self.COLORS["table_bg"],
                        font=("Tajawal", 14, "normal"),
                        borderwidth=1,
                        relief="solid")

        # تكوين رؤوس الأعمدة
        style.configure("Formal.Treeview.Heading",
                        font=("Tajawal", 16, "bold"),
                        background="#1E3A5F",  # أزرق داكن رسمي
                        foreground="#FFFFFF",
                        relief="raised",
                        borderwidth=1,
                        padding=[10, 8])

        # تكوين التحديد
        style.map('Formal.Treeview',
                  background=[('selected', '#4682B4'),  # أزرق رسمي
                              ('active', '#E6F2FF')],  # أزرق فاتح جداً
                  foreground=[('selected', '#FFFFFF'),
                              ('active', '#000000')])

        # شريط التمرير
        style.configure("Formal.Vertical.TScrollbar",
                        background="#D3D3D3",
                        troughcolor="#F5F5F5",
                        borderwidth=1,
                        arrowcolor="#696969",
                        width=16)

        # شريط التمرير العمودي
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", style="Formal.Vertical.TScrollbar")
        v_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # شريط التمرير الأفقي
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal")
        h_scrollbar.pack(side=tk.BOTTOM, fill=tk.X)

        # إنشاء الجدول - تم حذف عمود id
        self.teachers_tree = ttk.Treeview(
            table_frame,
            columns=("name", "rank", "id_number", "workplace", "qualification", "phone"),
            show="tree headings",
            style="Formal.Treeview",
            yscrollcommand=v_scrollbar.set,
            xscrollcommand=h_scrollbar.set,
            height=12
        )

        # إخفاء عمود الشجرة
        self.teachers_tree.column("#0", width=0, stretch=tk.NO)

        # تكوين الأعمدة - بدون عمود الرقم
        column_configs = [
            ("name", "الاسم الكامل", 350, tk.CENTER),
            ("rank", "الرتبة العسكرية", 220, tk.CENTER),
            ("id_number", "رقم الهوية", 200, tk.CENTER),
            ("workplace", "جهة العمل", 350, tk.CENTER),
            ("qualification", "المؤهل الدراسي", 280, tk.CENTER),
            ("phone", "رقم الجوال", 180, tk.CENTER)
        ]

        for col_id, heading, width, anchor in column_configs:
            self.teachers_tree.column(col_id, width=width, anchor=anchor, minwidth=width - 30)
            self.teachers_tree.heading(col_id, text=heading, anchor=tk.CENTER)

        # تكوين ألوان الصفوف
        self.teachers_tree.tag_configure('oddrow',
                                         background=self.COLORS["table_bg"],
                                         foreground='#2c2c2c',
                                         font=("Tajawal", 13, "normal"))
        self.teachers_tree.tag_configure('evenrow',
                                         background=self.COLORS["table_alt"],
                                         foreground='#2c2c2c',
                                         font=("Tajawal", 13, "normal"))

        self.teachers_tree.pack(fill=tk.BOTH, expand=True)
        v_scrollbar.config(command=self.teachers_tree.yview)
        h_scrollbar.config(command=self.teachers_tree.xview)

        # إطار المعلومات السفلي
        info_frame = tk.Frame(teachers_frame, bg="#1E3A5F", height=60)
        info_frame.pack(fill=tk.X, padx=15, pady=(5, 10))
        info_frame.pack_propagate(False)

        # خط فاصل علوي
        separator = tk.Frame(info_frame, bg=self.COLORS["card"], height=2)
        separator.pack(fill=tk.X)

        # إطار داخلي للمعلومات
        inner_info = tk.Frame(info_frame, bg="#1E3A5F")
        inner_info.pack(expand=True)

        self.teacher_count_label = tk.Label(
            inner_info,
            text="إجمالي المدرسين: 0 | منسوبي المدينة: 0 | المتعاونين: 0",
            font=("Tajawal", 14, "bold"),
            bg="#1E3A5F",
            fg="#FFFFFF"
        )
        self.teacher_count_label.pack(pady=15)

        # تحميل بيانات المدرسين
        self._load_teachers()

        # ربط الأحداث
        self.teachers_tree.bind("<Double-Button-1>", self._on_teacher_double_click)
        self.teachers_tree.bind("<Button-3>", self._show_context_menu)

    # إصلاحات لنظام إدارة هيئة التدريس

    # إصلاحات لنظام إدارة هيئة التدريس

    def _manage_course_teachers_paths(self):
        """إدارة مسارات هيئة التدريس - محدث مع زر مسؤولي المواد"""
        paths_window = tk.Toplevel(self)
        paths_window.title("مسارات هيئة التدريس")
        paths_window.geometry("1200x700")
        paths_window.configure(bg=self.COLORS["background"])

        # السماح بتكبير النافذة
        paths_window.resizable(True, True)
        paths_window.minsize(1000, 600)
        paths_window.state('normal')

        # توسيط النافذة
        paths_window.update_idletasks()
        x = (paths_window.winfo_screenwidth() - 1200) // 2
        y = (paths_window.winfo_screenheight() - 700) // 2
        paths_window.geometry(f"1200x700+{x}+{y}")

        # شريط العنوان
        header_frame = tk.Frame(paths_window, bg="#1E3A5F", height=80)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        header_content = tk.Frame(header_frame, bg="#1E3A5F")
        header_content.pack(expand=True, fill=tk.BOTH, padx=30)

        title_label = tk.Label(
            header_content,
            text="مسارات هيئة التدريس",
            font=("Tajawal", 24, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack(side=tk.LEFT, pady=20)

        # إطار الأزرار في الشريط العلوي
        header_buttons = tk.Frame(header_content, bg="#1E3A5F")
        header_buttons.pack(side=tk.RIGHT, pady=20)

        # زر إضافة دورة
        add_course_btn = tk.Button(
            header_buttons,
            text="إضافة دورة",
            font=("Tajawal", 13, "bold"),
            bg="#4CAF50",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        add_course_btn.pack(side=tk.LEFT, padx=(0, 10))

        # زر حذف دورة
        delete_course_btn = tk.Button(
            header_buttons,
            text="حذف دورة",
            font=("Tajawal", 13, "bold"),
            bg="#dc3545",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        delete_course_btn.pack(side=tk.LEFT, padx=(0, 10))

        # زر تصدير البيانات
        export_btn = tk.Button(
            header_buttons,
            text="تصدير بيانات هيئة التدريس",
            font=("Tajawal", 13, "bold"),
            bg="#FF9800",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        export_btn.pack(side=tk.LEFT)

        # إطار البحث
        search_frame = tk.Frame(paths_window, bg=self.COLORS["surface"], height=70)
        search_frame.pack(fill=tk.X, padx=15, pady=(0, 10))
        search_frame.pack_propagate(False)

        search_container = tk.Frame(search_frame, bg=self.COLORS["surface"])
        search_container.pack(side=tk.LEFT, padx=20, pady=20)

        tk.Label(
            search_container,
            text="البحث عن دورة:",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        search_entry = tk.Entry(
            search_container,
            font=("Tajawal", 13),
            width=30,
            bd=2,
            relief=tk.FLAT,
            highlightthickness=2,
            highlightcolor="#1E3A5F"
        )
        search_entry.pack(side=tk.LEFT)

        # زر مسؤولي مواد البرامج التدريبية - جديد
        subject_teachers_btn = tk.Button(
            search_frame,
            text="📚 مسؤولي مواد البرامج التدريبية",
            font=("Tajawal", 14, "bold"),
            bg="#9C27B0",  # لون بنفسجي مميز
            fg="white",
            padx=25,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._manage_subject_teachers
        )
        subject_teachers_btn.pack(side=tk.RIGHT, padx=20)

        # خط فاصل
        separator = tk.Frame(paths_window, bg=self.COLORS["border"], height=2)
        separator.pack(fill=tk.X, padx=15, pady=(0, 10))

        # إطار الجدول الرئيسي
        main_table_frame = tk.Frame(paths_window, bg=self.COLORS["table_bg"], bd=2, relief=tk.RIDGE)
        main_table_frame.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 10))

        table_frame = tk.Frame(main_table_frame, bg=self.COLORS["table_bg"])
        table_frame.pack(fill=tk.BOTH, expand=True, padx=3, pady=3)

        # شريط التمرير
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical")
        v_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # تنسيق Treeview
        style = ttk.Style()
        style.configure("Paths.Treeview",
                        background="#FFFFFF",
                        foreground="#000000",
                        rowheight=45,
                        fieldbackground="#FFFFFF",
                        font=("Tajawal", 14, "normal"),
                        borderwidth=1,
                        relief="solid")

        style.configure("Paths.Treeview.Heading",
                        font=("Tajawal", 16, "bold"),
                        background="#1E3A5F",
                        foreground="#FFFFFF",
                        relief="raised",
                        borderwidth=1,
                        padding=[10, 8])

        style.map('Paths.Treeview',
                  background=[('selected', '#4682B4'),
                              ('active', '#E6F2FF')],
                  foreground=[('selected', '#FFFFFF'),
                              ('active', '#000000')])

        # إنشاء الجدول
        paths_tree = ttk.Treeview(
            table_frame,
            columns=("course_name", "responsible_teacher", "substitute_teachers", "total"),
            show="tree headings",
            style="Paths.Treeview",
            yscrollcommand=v_scrollbar.set,
            height=12
        )

        # إخفاء عمود الشجرة
        paths_tree.column("#0", width=0, stretch=tk.NO)

        # تكوين الأعمدة
        column_configs = [
            ("course_name", "اسم الدورة", 400, tk.CENTER),
            ("responsible_teacher", "رئيس البرنامج التدريبي", 250, tk.CENTER),
            ("substitute_teachers", "هيئة التدريس", 350, tk.CENTER),
            ("total", "العدد الكلي", 120, tk.CENTER)
        ]

        for col_id, heading, width, anchor in column_configs:
            paths_tree.column(col_id, width=width, anchor=anchor, minwidth=width - 30)
            paths_tree.heading(col_id, text=heading, anchor=tk.CENTER)

        # تكوين ألوان الصفوف
        paths_tree.tag_configure('oddrow',
                                 background='#FFFFFF',
                                 font=("Tajawal", 13, "normal"))
        paths_tree.tag_configure('evenrow',
                                 background='#F0F8FF',
                                 font=("Tajawal", 13, "normal"))

        paths_tree.pack(fill=tk.BOTH, expand=True)
        v_scrollbar.config(command=paths_tree.yview)

        # دالة تحميل البيانات
        def load_course_paths(search_term=""):
            # مسح الجدول
            for item in paths_tree.get_children():
                paths_tree.delete(item)

            try:
                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات من course_names
                if search_term:
                    cursor.execute("""
                        SELECT DISTINCT name 
                        FROM course_names 
                        WHERE is_active = 1 AND LOWER(name) LIKE LOWER(?)
                        ORDER BY name
                    """, (f'%{search_term}%',))
                else:
                    cursor.execute("""
                        SELECT DISTINCT name 
                        FROM course_names 
                        WHERE is_active = 1
                        ORDER BY name
                    """)

                all_courses = cursor.fetchall()

                for index, (course_name,) in enumerate(all_courses):
                    # الحصول على المدرس المسؤول
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                    """, (course_name,))

                    responsible = cursor.fetchone()
                    responsible_name = responsible[0] if responsible else "غير محدد"

                    # الحصول على هيئة التدريس - منسوبي المدينة
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category = 'منسوبي المدينة'
                        ORDER BY t.name
                    """, (course_name,))

                    city_staff = cursor.fetchall()

                    # الحصول على هيئة التدريس - المتعاونين
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY t.name
                    """, (course_name,))

                    collaborators = cursor.fetchall()

                    # تجميع الأعضاء
                    all_members = []
                    if city_staff:
                        all_members.append(
                            f"منسوبي المدينة ({len(city_staff)}): " + ", ".join([s[0] for s in city_staff]))
                    if collaborators:
                        all_members.append(
                            f"المتعاونين ({len(collaborators)}): " + ", ".join([c[0] for c in collaborators]))

                    substitute_names = " | ".join(all_members) if all_members else "لا يوجد"

                    # العدد الكلي
                    total_count = len(city_staff) + len(collaborators) + (1 if responsible else 0)

                    # تحديد التاج
                    tag = 'evenrow' if index % 2 == 0 else 'oddrow'

                    # إضافة للجدول
                    paths_tree.insert("", tk.END, values=(
                        course_name,
                        responsible_name,
                        substitute_names,
                        total_count
                    ), tags=(tag,))

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ في تحميل البيانات: {str(e)}")

        # دالة إضافة دورة جديدة
        def add_course_to_paths():
            """إضافة دورة جديدة لمسارات هيئة التدريس"""
            add_dialog = tk.Toplevel(paths_window)
            add_dialog.title("إضافة دورة")
            add_dialog.geometry("600x500")
            add_dialog.configure(bg=self.COLORS["surface"])
            add_dialog.transient(paths_window)
            add_dialog.grab_set()

            # توسيط النافذة
            add_dialog.update_idletasks()
            x = (add_dialog.winfo_screenwidth() - 600) // 2
            y = (add_dialog.winfo_screenheight() - 500) // 2
            add_dialog.geometry(f"600x500+{x}+{y}")

            # العنوان
            tk.Label(
                add_dialog,
                text="إضافة دورة لهيئة التدريس",
                font=("Tajawal", 16, "bold"),
                bg=self.COLORS["primary"],
                fg="white",
                pady=20
            ).pack(fill=tk.X)

            # إطار المحتوى
            content = tk.Frame(add_dialog, bg=self.COLORS["surface"], padx=30, pady=20)
            content.pack(fill=tk.BOTH, expand=True)

            # البحث
            tk.Label(
                content,
                text="البحث عن دورة:",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["surface"]
            ).pack(anchor=tk.W, pady=(0, 10))

            search_frame = tk.Frame(content, bg=self.COLORS["surface"])
            search_frame.pack(fill=tk.X, pady=(0, 20))

            search_entry = tk.Entry(
                search_frame,
                font=("Tajawal", 12),
                bd=2,
                relief=tk.FLAT,
                highlightthickness=2,
                highlightcolor=self.COLORS["primary"]
            )
            search_entry.pack(fill=tk.X)

            # قائمة الدورات
            list_frame = tk.Frame(content, bg=self.COLORS["surface"], relief=tk.GROOVE, bd=2)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame)
            scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

            courses_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 12),
                yscrollcommand=scrollbar.set,
                selectmode=tk.SINGLE
            )
            courses_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=courses_listbox.yview)

            # متغير لتخزين جميع الدورات المتاحة
            all_available_courses = []
            course_names_map = {}  # لربط الأسماء بالمعرفات

            # تحميل الدورات المتاحة
            def load_available_courses():
                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات من course_names
                cursor.execute("""
                    SELECT DISTINCT name 
                    FROM course_names 
                    WHERE is_active = 1 
                    ORDER BY name
                """)

                all_courses = cursor.fetchall()

                # الحصول على الدورات التي لها مدرسين بالفعل
                cursor.execute("""
                    SELECT DISTINCT course_name 
                    FROM course_teacher_paths
                """)
                existing_courses = set([c[0] for c in cursor.fetchall()])

                # تحديد الدورات المتاحة (غير الموجودة في مسارات هيئة التدريس)
                all_available_courses.clear()
                course_names_map.clear()

                for i, (course_name,) in enumerate(all_courses):
                    if course_name not in existing_courses:
                        all_available_courses.append(course_name)
                        course_names_map[i] = course_name

                # عرض جميع الدورات المتاحة
                update_course_list("")

            def update_course_list(search_text):
                """تحديث قائمة الدورات بناءً على البحث"""
                courses_listbox.delete(0, tk.END)

                search_text = search_text.strip().lower()

                for course_name in all_available_courses:
                    if not search_text or search_text in course_name.lower():
                        courses_listbox.insert(tk.END, course_name)

            # ربط البحث
            def on_search_change(event=None):
                update_course_list(search_entry.get())

            search_entry.bind('<KeyRelease>', on_search_change)

            # إطار الأزرار
            btn_frame = tk.Frame(add_dialog, bg=self.COLORS["surface"])
            btn_frame.pack(fill=tk.X, pady=20)

            def confirm_add():
                selection = courses_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار دورة")
                    return

                # الحصول على اسم الدورة المحددة فقط
                selected_course_name = courses_listbox.get(selection[0])

                try:
                    cursor = self.db_conn.cursor()

                    # التحقق مرة أخرى من أن الدورة غير موجودة
                    cursor.execute("""
                        SELECT COUNT(*) FROM course_teacher_paths 
                        WHERE course_name = ?
                    """, (selected_course_name,))

                    if cursor.fetchone()[0] > 0:
                        messagebox.showwarning("تنبيه", "هذه الدورة موجودة بالفعل في هيئة التدريس")
                        return

                    # إغلاق النافذة
                    add_dialog.destroy()

                    # عرض رسالة النجاح
                    messagebox.showinfo("نجاح",
                                        f"تمت إضافة دورة '{selected_course_name}' لهيئة التدريس\n"
                                        "يمكنك الآن تعيين رئيس البرنامج التدريبي والأعضاء")

                    # تحديث القائمة الرئيسية
                    load_course_paths()

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            tk.Button(
                btn_frame,
                text="إضافة",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["success"],
                fg="white",
                bd=0,
                padx=30,
                pady=8,
                cursor="hand2",
                command=confirm_add
            ).pack(side=tk.LEFT, padx=20)

            tk.Button(
                btn_frame,
                text="إلغاء",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["danger"],
                fg="white",
                bd=0,
                padx=30,
                pady=8,
                cursor="hand2",
                command=add_dialog.destroy
            ).pack(side=tk.RIGHT, padx=20)

            # تحميل الدورات الأولية
            load_available_courses()

        # دالة حذف دورة من هيئة التدريس
        def delete_course_from_paths():
            """حذف دورة من مسارات هيئة التدريس مع جميع أعضائها"""
            selection = paths_tree.selection()
            if not selection:
                messagebox.showinfo("تنبيه", "يرجى اختيار دورة لحذفها")
                return

            item = paths_tree.item(selection[0])
            course_name = item['values'][0]

            # التأكيد
            result = messagebox.askyesno(
                "تأكيد الحذف",
                f"هل تريد حذف دورة '{course_name}' من هيئة التدريس؟\n\n"
                "سيتم حذف:\n"
                "• رئيس البرنامج التدريبي\n"
                "• جميع أعضاء هيئة التدريس\n\n"
                "ملاحظة: لن يتم حذف الدورة من مسميات الدورات"
            )

            if result:
                try:
                    cursor = self.db_conn.cursor()

                    # حذف جميع السجلات المرتبطة بهذه الدورة من course_teacher_paths
                    cursor.execute("""
                        DELETE FROM course_teacher_paths 
                        WHERE course_name = ?
                    """, (course_name,))

                    self.db_conn.commit()

                    # حذف العنصر من الشجرة مباشرة
                    paths_tree.delete(selection[0])

                    messagebox.showinfo("نجاح", f"تم حذف دورة '{course_name}' من هيئة التدريس بنجاح")

                except Exception as e:
                    self.db_conn.rollback()
                    messagebox.showerror("خطأ", f"حدث خطأ أثناء الحذف: {str(e)}")

        # دالة إدارة مدرسي الدورة
        def manage_course_teachers(event=None):
            """إدارة مدرسي الدورة مع قسمين منفصلين لمنسوبي المدينة والمتعاونين"""
            selection = paths_tree.selection()
            if not selection:
                return

            item = paths_tree.item(selection[0])
            course_name = item['values'][0]

            # نافذة إدارة مدرسي الدورة
            manage_window = tk.Toplevel(paths_window)
            manage_window.title(f"إدارة هيئة التدريس - {course_name}")
            manage_window.geometry("1200x750")
            manage_window.configure(bg="#F5F5F5")
            manage_window.resizable(True, True)
            manage_window.minsize(1100, 700)

            # توسيط النافذة
            manage_window.update_idletasks()
            x = (manage_window.winfo_screenwidth() - 1200) // 2
            y = (manage_window.winfo_screenheight() - 750) // 2
            manage_window.geometry(f"1200x750+{x}+{y}")

            # شريط العنوان الرئيسي
            header_frame = tk.Frame(manage_window, bg="#1E3A5F", height=70)
            header_frame.pack(fill=tk.X)
            header_frame.pack_propagate(False)

            header_content = tk.Frame(header_frame, bg="#1E3A5F")
            header_content.pack(expand=True)

            tk.Label(
                header_content,
                text="إدارة هيئة التدريس",
                font=("Tajawal", 20, "bold"),
                bg="#1E3A5F",
                fg="white"
            ).pack()

            tk.Label(
                header_content,
                text=f"دورة: {course_name}",
                font=("Tajawal", 14),
                bg="#1E3A5F",
                fg="#E0E0E0"
            ).pack()

            # إطار المحتوى الرئيسي
            main_container = tk.Frame(manage_window, bg="#F5F5F5")
            main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=15)

            # القسم الأول: المدرس المسؤول
            responsible_frame = tk.Frame(main_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            responsible_frame.pack(fill=tk.X, pady=(0, 15))

            # رأس قسم المسؤول
            resp_header = tk.Frame(responsible_frame, bg="#2C3E50", height=40)
            resp_header.pack(fill=tk.X)
            resp_header.pack_propagate(False)

            tk.Label(
                resp_header,
                text=f"رئيس البرنامج التدريبي لـ ({course_name})",
                font=("Tajawal", 14, "bold"),
                bg="#2C3E50",
                fg="white"
            ).pack(expand=True)

            # محتوى قسم المسؤول
            resp_content = tk.Frame(responsible_frame, bg="#FFFFFF", padx=20, pady=15)
            resp_content.pack(fill=tk.BOTH)

            # تقسيم أفقي
            resp_content.grid_columnconfigure(0, weight=1)
            resp_content.grid_columnconfigure(1, weight=1)

            # الجانب الأيمن - المسؤول الحالي
            right_resp = tk.Frame(resp_content, bg="#FFFFFF")
            right_resp.grid(row=0, column=1, sticky="nsew", padx=(10, 0))

            tk.Label(
                right_resp,
                text="الرئيس الحالي",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # إطار عرض المسؤول
            current_resp_frame = tk.Frame(right_resp, bg="#F8F9FA", relief=tk.RIDGE, bd=1, height=70)
            current_resp_frame.pack(fill=tk.X)
            current_resp_frame.pack_propagate(False)

            current_responsible_label = tk.Label(
                current_resp_frame,
                text="",
                font=("Tajawal", 12),
                bg="#F8F9FA",
                fg="#333333",
                justify=tk.CENTER
            )
            current_responsible_label.pack(expand=True)

            # زر حذف المسؤول
            remove_resp_btn = tk.Button(
                right_resp,
                text="حذف الرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_resp_btn.pack(pady=(10, 0))

            # الجانب الأيسر - البحث وتعيين
            left_resp = tk.Frame(resp_content, bg="#FFFFFF")
            left_resp.grid(row=0, column=0, sticky="nsew", padx=(0, 10))

            tk.Label(
                left_resp,
                text="تعيين رئيس جديد",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # البحث
            search_container = tk.Frame(left_resp, bg="#FFFFFF")
            search_container.pack(fill=tk.X, pady=(0, 8))

            tk.Label(
                search_container,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            responsible_search_entry = tk.Entry(
                search_container,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1E3A5F"
            )
            responsible_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث
            list_frame = tk.Frame(left_resp, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame, width=12)
            scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            responsible_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 11),
                height=4,
                bd=0,
                highlightthickness=0,
                yscrollcommand=scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#E3F2FD",
                selectforeground="#0D47A1"
            )
            responsible_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=responsible_listbox.yview)

            # زر التعيين
            assign_resp_btn = tk.Button(
                left_resp,
                text="تعيين كرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#3498DB",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            assign_resp_btn.pack(pady=(8, 0))

            # القسم الثاني: أعضاء هيئة التدريس - مقسم إلى قسمين
            members_container = tk.Frame(main_container, bg="#F5F5F5")
            members_container.pack(fill=tk.BOTH, expand=True)

            # القسم الأيمن - منسوبي المدينة
            city_staff_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            city_staff_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(5, 0))

            # رأس قسم منسوبي المدينة - لون أزرق داكن رسمي
            city_header = tk.Frame(city_staff_frame, bg="#1B4F72", height=40)
            city_header.pack(fill=tk.X)
            city_header.pack_propagate(False)

            city_header_content = tk.Frame(city_header, bg="#1B4F72")
            city_header_content.pack(expand=True)

            tk.Label(
                city_header_content,
                text="أعضاء هيئة التدريس - منسوبي المدينة",
                font=("Tajawal", 14, "bold"),
                bg="#1B4F72",
                fg="white"
            ).pack(side=tk.RIGHT)

            city_count_label = tk.Label(
                city_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#1B4F72",
                fg="white"
            )
            city_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم منسوبي المدينة
            city_content = tk.Frame(city_staff_frame, bg="#FFFFFF", padx=15, pady=15)
            city_content.pack(fill=tk.BOTH, expand=True)

            # البحث في منسوبي المدينة
            city_search_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                city_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            city_search_entry = tk.Entry(
                city_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1B4F72"
            )
            city_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث لمنسوبي المدينة
            city_search_list_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            city_search_scrollbar = tk.Scrollbar(city_search_list_frame, width=12)
            city_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_search_listbox = tk.Listbox(
                city_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D4E6F1",
                selectforeground="#1B4F72"
            )
            city_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_search_scrollbar.config(command=city_search_listbox.yview)

            # إطار الأزرار لمنسوبي المدينة
            city_buttons_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_buttons_frame.pack(pady=(5, 10))

            # زر إضافة منسوب مدينة
            add_city_btn = tk.Button(
                city_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#2874A6",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_city_btn.pack(side=tk.RIGHT, padx=(0, 10))

            # زر حذف منسوب مدينة من البحث
            delete_city_search_btn = tk.Button(
                city_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            delete_city_search_btn.pack(side=tk.RIGHT)

            # خط فاصل
            tk.Frame(city_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من منسوبي المدينة
            tk.Label(
                city_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#1B4F72"
            ).pack(anchor=tk.E, pady=(0, 10))

            city_members_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_members_frame.pack(fill=tk.BOTH, expand=True)

            city_members_scrollbar = tk.Scrollbar(city_members_frame, width=12)
            city_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_members_listbox = tk.Listbox(
                city_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_members_scrollbar.set,
                bg="#EBF5FB",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            city_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_members_scrollbar.config(command=city_members_listbox.yview)

            # القسم الأيسر - المتعاونين
            collaborators_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            collaborators_frame.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 5))

            # رأس قسم المتعاونين - لون رمادي داكن رسمي
            collab_header = tk.Frame(collaborators_frame, bg="#424949", height=40)
            collab_header.pack(fill=tk.X)
            collab_header.pack_propagate(False)

            collab_header_content = tk.Frame(collab_header, bg="#424949")
            collab_header_content.pack(expand=True)

            tk.Label(
                collab_header_content,
                text="أعضاء هيئة التدريس - المتعاونين",
                font=("Tajawal", 14, "bold"),
                bg="#424949",
                fg="white"
            ).pack(side=tk.RIGHT)

            collab_count_label = tk.Label(
                collab_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#424949",
                fg="white"
            )
            collab_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم المتعاونين
            collab_content = tk.Frame(collaborators_frame, bg="#FFFFFF", padx=15, pady=15)
            collab_content.pack(fill=tk.BOTH, expand=True)

            # البحث في المتعاونين
            collab_search_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                collab_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            collab_search_entry = tk.Entry(
                collab_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#424949"
            )
            collab_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث للمتعاونين
            collab_search_list_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            collab_search_scrollbar = tk.Scrollbar(collab_search_list_frame, width=12)
            collab_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_search_listbox = tk.Listbox(
                collab_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D5DBDB",
                selectforeground="#212F3C"
            )
            collab_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_search_scrollbar.config(command=collab_search_listbox.yview)

            # إطار الأزرار للمتعاونين
            collab_buttons_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_buttons_frame.pack(pady=(5, 10))

            # زر إضافة متعاون
            add_collab_btn = tk.Button(
                collab_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#5D6D7E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_collab_btn.pack(side=tk.RIGHT, padx=(0, 10))

            # زر حذف متعاون من البحث
            delete_collab_search_btn = tk.Button(
                collab_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            delete_collab_search_btn.pack(side=tk.RIGHT)

            # خط فاصل
            tk.Frame(collab_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من المتعاونين
            tk.Label(
                collab_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#424949"
            ).pack(anchor=tk.E, pady=(0, 10))

            collab_members_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_members_frame.pack(fill=tk.BOTH, expand=True)

            collab_members_scrollbar = tk.Scrollbar(collab_members_frame, width=12)
            collab_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_members_listbox = tk.Listbox(
                collab_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_members_scrollbar.set,
                bg="#F4F6F6",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            collab_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_members_scrollbar.config(command=collab_members_listbox.yview)

            # الشريط السفلي
            footer_frame = tk.Frame(manage_window, bg="#1E3A5F", height=50)
            footer_frame.pack(fill=tk.X)
            footer_frame.pack_propagate(False)

            close_btn = tk.Button(
                footer_frame,
                text="إغلاق",
                font=("Tajawal", 13, "bold"),
                bg="#34495E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=35,
                pady=10,
                command=lambda: [manage_window.destroy(), load_course_paths(search_entry.get())]
            )
            close_btn.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            # maps لتخزين معرفات المدرسين
            manage_window.city_members_map = {}
            manage_window.collab_members_map = {}
            manage_window.city_search_map = {}
            manage_window.collab_search_map = {}
            manage_window.responsible_map = {}

            # تحميل البيانات الحالية
            def load_current_data():
                cursor = self.db_conn.cursor()

                # تحميل المدرس المسؤول
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                """, (course_name,))

                responsible = cursor.fetchone()
                if responsible:
                    current_responsible_label.config(
                        text=f"{responsible[1]}\n{responsible[2]} - {responsible[3]}",
                        font=("Tajawal", 11, "bold")
                    )
                    remove_resp_btn.config(state=tk.NORMAL)
                else:
                    current_responsible_label.config(
                        text="لم يتم تعيين مسؤول بعد",
                        fg="#999999",
                        font=("Tajawal", 11, "italic")
                    )
                    remove_resp_btn.config(state=tk.DISABLED)

                # تحميل أعضاء هيئة التدريس - منسوبي المدينة
                city_members_listbox.delete(0, tk.END)
                manage_window.city_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (course_name,))

                city_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace) in enumerate(city_members):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_members_listbox.insert(tk.END, display_text)
                    manage_window.city_members_map[i] = teacher_id

                city_count_label.config(text=f"العدد: {len(city_members)}")

                # تحميل أعضاء هيئة التدريس - المتعاونين
                collab_members_listbox.delete(0, tk.END)
                manage_window.collab_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (course_name,))

                collab_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(collab_members):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_members_listbox.insert(tk.END, display_text)
                    manage_window.collab_members_map[i] = teacher_id

                collab_count_label.config(text=f"العدد: {len(collab_members)}")

            # البحث عن المدرسين للمسؤول (يبحث في الجميع)
            def search_responsible_teachers(*args):
                search_term = responsible_search_entry.get().strip()
                responsible_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT id, name, rank, workplace, category
                    FROM teachers 
                    WHERE LOWER(name) LIKE LOWER(?)
                    ORDER BY name
                    LIMIT 20
                """, (f'%{search_term}%',))

                teachers = cursor.fetchall()
                manage_window.responsible_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    if category:
                        display_text += f" [{category}]"
                    responsible_listbox.insert(tk.END, display_text)
                    manage_window.responsible_map[i] = teacher_id

            # البحث في منسوبي المدينة
            def search_city_teachers(*args):
                search_term = city_search_entry.get().strip()
                city_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category = 'منسوبي المدينة'
                    AND t.id NOT IN (
                        SELECT teacher_id FROM course_teacher_paths 
                        WHERE course_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name))

                teachers = cursor.fetchall()
                manage_window.city_search_map.clear()
                for i, (teacher_id, name, rank, workplace) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_search_listbox.insert(tk.END, display_text)
                    manage_window.city_search_map[i] = teacher_id

            # البحث في المتعاونين
            def search_collab_teachers(*args):
                search_term = collab_search_entry.get().strip()
                collab_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    AND t.id NOT IN (
                        SELECT teacher_id FROM course_teacher_paths 
                        WHERE course_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name))

                teachers = cursor.fetchall()
                manage_window.collab_search_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_search_listbox.insert(tk.END, display_text)
                    manage_window.collab_search_map[i] = teacher_id

            # تعيين مدرس مسؤول
            def assign_responsible():
                selection = responsible_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.responsible_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن تعيين هذا المدرس كرئيس للبرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                if messagebox.askyesno("تأكيد", "هل تريد تعيين هذا المدرس كرئيس للبرنامج التدريبي؟"):
                    try:
                        cursor = self.db_conn.cursor()

                        # إلغاء أي مسؤول سابق
                        cursor.execute("""
                            UPDATE course_teacher_paths 
                            SET is_responsible = 0 
                            WHERE course_name = ? AND is_responsible = 1
                        """, (course_name,))

                        # تعيين المسؤول الجديد
                        cursor.execute("""
                            INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                            VALUES (?, ?, 1, ?)
                            ON CONFLICT(course_name, teacher_id) 
                            DO UPDATE SET is_responsible = 1
                        """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                        self.db_conn.commit()

                        load_current_data()
                        responsible_search_entry.delete(0, tk.END)
                        responsible_listbox.delete(0, tk.END)

                        messagebox.showinfo("نجاح", "تم تعيين رئيس البرنامج التدريبي بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف المسؤول
            def remove_responsible():
                if messagebox.askyesno("تأكيد", "هل تريد حذف المسؤول الحالي؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND is_responsible = 1
                        """, (course_name,))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف المسؤول بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة منسوب مدينة
            def add_city_member():
                selection = city_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.city_search_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن إضافة هذا المدرس كعضو في البرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    city_search_entry.delete(0, tk.END)
                    city_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة متعاون
            def add_collab_member():
                selection = collab_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.collab_search_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن إضافة هذا المدرس كعضو في البرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    collab_search_entry.delete(0, tk.END)
                    collab_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف منسوب مدينة من البحث
            def remove_city_member_from_search():
                selection = city_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو من قائمة الأعضاء الحاليين للحذف")
                    return

                teacher_id = manage_window.city_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من هيئة التدريس؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف العضو بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف متعاون من البحث
            def remove_collab_member_from_search():
                selection = collab_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو من قائمة الأعضاء الحاليين للحذف")
                    return

                teacher_id = manage_window.collab_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من هيئة التدريس؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف العضو بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # ربط الأحداث
            responsible_search_entry.bind('<KeyRelease>', search_responsible_teachers)
            city_search_entry.bind('<KeyRelease>', search_city_teachers)
            collab_search_entry.bind('<KeyRelease>', search_collab_teachers)

            assign_resp_btn.config(command=assign_responsible)
            remove_resp_btn.config(command=remove_responsible)
            add_city_btn.config(command=add_city_member)
            add_collab_btn.config(command=add_collab_member)
            delete_city_search_btn.config(command=remove_city_member_from_search)
            delete_collab_search_btn.config(command=remove_collab_member_from_search)

            # تحميل البيانات الأولية
            load_current_data()

        # دالة تصدير البيانات - محدثة مع مسؤولي المواد
        def export_faculty_data():
            """تصدير بيانات هيئة التدريس مع مسؤولي المواد"""
            if not EXCEL_AVAILABLE:
                messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
                return

            try:
                from tkinter import filedialog
                import pandas as pd
                from openpyxl import Workbook
                from openpyxl.styles import Alignment, Font, PatternFill, Border, Side
                from openpyxl.utils import get_column_letter

                # اختيار مكان الحفظ
                file_path = filedialog.asksaveasfilename(
                    title="حفظ بيانات هيئة التدريس",
                    defaultextension=".xlsx",
                    initialfile=f"هيئة_التدريس_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                    filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
                )

                if not file_path:
                    return

                cursor = self.db_conn.cursor()

                # إنشاء ملف Excel
                wb = Workbook()

                # الورقة الأولى: هيئة التدريس
                ws1 = wb.active
                ws1.title = "هيئة التدريس"

                # تعيين اتجاه الورقة من اليمين لليسار
                ws1.sheet_view.rightToLeft = True

                # العناوين
                headers1 = ["م", "اسم الدورة", "رئيس البرنامج التدريبي", "منسوبي المدينة", "المتعاونين", "العدد الكلي"]

                # كتابة العناوين
                for col, header in enumerate(headers1, 1):
                    cell = ws1.cell(row=1, column=col, value=header)
                    cell.font = Font(name='Arial', size=14, bold=True, color="FFFFFF")
                    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
                    cell.fill = PatternFill(start_color="1E3A5F", end_color="1E3A5F", fill_type="solid")

                # تنسيق الحدود
                thin_border = Border(
                    left=Side(style='thin'),
                    right=Side(style='thin'),
                    top=Side(style='thin'),
                    bottom=Side(style='thin')
                )

                # الحصول على جميع الدورات
                cursor.execute("""
                    SELECT DISTINCT course_name 
                    FROM course_teacher_paths 
                    ORDER BY course_name
                """)
                courses = cursor.fetchall()

                # كتابة بيانات هيئة التدريس
                row_num = 2
                for idx, (course_name,) in enumerate(courses, 1):
                    # المسؤول
                    cursor.execute("""
                        SELECT t.name, t.rank 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                    """, (course_name,))
                    responsible = cursor.fetchone()
                    responsible_text = f"{responsible[0]} - {responsible[1]}" if responsible else "غير محدد"

                    # منسوبي المدينة
                    cursor.execute("""
                        SELECT t.name, t.rank 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category = 'منسوبي المدينة'
                        ORDER BY t.name
                    """, (course_name,))
                    city_staff = cursor.fetchall()
                    city_staff_text = "\n".join([f"{m[0]} - {m[1]}" for m in city_staff]) if city_staff else "لا يوجد"

                    # المتعاونين
                    cursor.execute("""
                        SELECT t.name, t.rank, t.category
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY t.name
                    """, (course_name,))
                    collaborators = cursor.fetchall()
                    collab_text = "\n".join(
                        [f"{c[0]} - {c[1]} ({c[2]})" for c in collaborators]) if collaborators else "لا يوجد"

                    # العدد الكلي
                    total = len(city_staff) + len(collaborators) + (1 if responsible else 0)

                    # كتابة الصف
                    data_row = [idx, course_name, responsible_text, city_staff_text, collab_text, total]
                    for col, value in enumerate(data_row, 1):
                        cell = ws1.cell(row=row_num, column=col, value=value)
                        cell.font = Font(name='Arial', size=12)
                        cell.alignment = Alignment(
                            horizontal='right' if col > 1 else 'center',
                            vertical='center',
                            wrap_text=True
                        )
                        cell.border = thin_border

                        # تلوين الصفوف بالتناوب
                        if row_num % 2 == 0:
                            cell.fill = PatternFill(start_color="F0F0F0", end_color="F0F0F0", fill_type="solid")

                    row_num += 1

                # تعيين عرض الأعمدة
                ws1.column_dimensions['A'].width = 8  # م
                ws1.column_dimensions['B'].width = 40  # اسم الدورة
                ws1.column_dimensions['C'].width = 35  # المسؤول
                ws1.column_dimensions['D'].width = 50  # منسوبي المدينة
                ws1.column_dimensions['E'].width = 50  # المتعاونين
                ws1.column_dimensions['F'].width = 15  # العدد الكلي

                # الورقة الثانية: مسؤولي المواد
                ws2 = wb.create_sheet("مسؤولي المواد")
                ws2.sheet_view.rightToLeft = True

                # عناوين ورقة مسؤولي المواد
                headers2 = ["م", "اسم الدورة", "اسم المادة", "رئيس المادة", "أعضاء منسوبي المدينة", "أعضاء متعاونين",
                            "العدد"]

                # كتابة العناوين
                for col, header in enumerate(headers2, 1):
                    cell = ws2.cell(row=1, column=col, value=header)
                    cell.font = Font(name='Arial', size=14, bold=True, color="FFFFFF")
                    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
                    cell.fill = PatternFill(start_color="9C27B0", end_color="9C27B0", fill_type="solid")

                # الحصول على جميع الدورات والمواد
                cursor.execute("""
                    SELECT DISTINCT cn.name, cds.subject_name
                    FROM course_names cn
                    INNER JOIN course_default_subjects cds ON cn.id = cds.course_name_id
                    WHERE cn.is_active = 1
                    ORDER BY cn.name, cds.subject_order
                """)

                course_subjects = cursor.fetchall()
                row_num2 = 2

                for idx, (course_name, subject_name) in enumerate(course_subjects, 1):
                    # رئيس المادة
                    cursor.execute("""
                        SELECT t.name, t.rank
                        FROM subject_teachers st
                        JOIN teachers t ON st.teacher_id = t.id
                        WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 1
                    """, (course_name, subject_name))
                    responsible = cursor.fetchone()
                    responsible_text = f"{responsible[0]} - {responsible[1]}" if responsible else "غير محدد"

                    # أعضاء منسوبي المدينة
                    cursor.execute("""
                        SELECT t.name, t.rank
                        FROM subject_teachers st
                        JOIN teachers t ON st.teacher_id = t.id
                        WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                        AND t.category = 'منسوبي المدينة'
                        ORDER BY t.name
                    """, (course_name, subject_name))
                    city_members = cursor.fetchall()
                    city_text = "\n".join([f"{m[0]} - {m[1]}" for m in city_members]) if city_members else "لا يوجد"

                    # أعضاء متعاونين
                    cursor.execute("""
                        SELECT t.name, t.rank, t.category
                        FROM subject_teachers st
                        JOIN teachers t ON st.teacher_id = t.id
                        WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                        AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY t.name
                    """, (course_name, subject_name))
                    collab_members = cursor.fetchall()
                    collab_text = "\n".join(
                        [f"{c[0]} - {c[1]} ({c[2]})" for c in collab_members]) if collab_members else "لا يوجد"

                    # العدد الكلي
                    total = len(city_members) + len(collab_members) + (1 if responsible else 0)

                    # كتابة الصف
                    data_row2 = [idx, course_name, subject_name, responsible_text, city_text, collab_text, total]
                    for col, value in enumerate(data_row2, 1):
                        cell = ws2.cell(row=row_num2, column=col, value=value)
                        cell.font = Font(name='Arial', size=12)
                        cell.alignment = Alignment(
                            horizontal='right' if col > 1 else 'center',
                            vertical='center',
                            wrap_text=True
                        )
                        cell.border = thin_border

                        # تلوين الصفوف بالتناوب
                        if row_num2 % 2 == 0:
                            cell.fill = PatternFill(start_color="F3E5F5", end_color="F3E5F5", fill_type="solid")

                    row_num2 += 1

                # تعيين عرض الأعمدة للورقة الثانية
                ws2.column_dimensions['A'].width = 8  # م
                ws2.column_dimensions['B'].width = 35  # اسم الدورة
                ws2.column_dimensions['C'].width = 35  # اسم المادة
                ws2.column_dimensions['D'].width = 30  # رئيس المادة
                ws2.column_dimensions['E'].width = 45  # أعضاء منسوبي المدينة
                ws2.column_dimensions['F'].width = 45  # أعضاء متعاونين
                ws2.column_dimensions['G'].width = 10  # العدد

                # حفظ الملف
                wb.save(file_path)
                messagebox.showinfo("نجاح", f"تم تصدير البيانات بنجاح\n{file_path}")

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

        # ربط الأزرار
        add_course_btn.config(command=add_course_to_paths)
        delete_course_btn.config(command=delete_course_from_paths)
        export_btn.config(command=export_faculty_data)

        # ربط النقر على الجدول
        paths_tree.bind("<Double-Button-1>", manage_course_teachers)

        # البحث الديناميكي
        def dynamic_search(*args):
            search_term = search_entry.get().strip()
            load_course_paths(search_term)

        # ربط البحث الديناميكي
        search_entry.bind('<KeyRelease>', dynamic_search)

        # تسمية للمساعدة
        help_label = tk.Label(
            paths_window,
            text="💡 انقر مرتين على أي دورة لإدارة هيئة التدريس",
            font=("Tajawal", 12),
            bg=self.COLORS["background"],
            fg="#666"
        )
        help_label.pack(pady=5)

        # زر الإغلاق
        close_btn = tk.Button(
            paths_window,
            text="إغلاق",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["dark"],
            fg="white",
            bd=0,
            padx=30,
            pady=12,
            cursor="hand2",
            command=paths_window.destroy
        )
        close_btn.pack(pady=10)

        # تحميل البيانات
        load_course_paths()

    def _manage_subject_teachers(self):
        """إدارة مسؤولي مواد البرامج التدريبية"""
        subjects_window = tk.Toplevel(self)
        subjects_window.title("مسؤولي مواد البرامج التدريبية")
        subjects_window.geometry("1200x700")
        subjects_window.configure(bg=self.COLORS["background"])

        # السماح بتكبير النافذة
        subjects_window.resizable(True, True)
        subjects_window.minsize(1000, 600)
        subjects_window.state('normal')

        # توسيط النافذة
        subjects_window.update_idletasks()
        x = (subjects_window.winfo_screenwidth() - 1200) // 2
        y = (subjects_window.winfo_screenheight() - 700) // 2
        subjects_window.geometry(f"1200x700+{x}+{y}")

        # شريط العنوان
        header_frame = tk.Frame(subjects_window, bg="#1E3A5F", height=80)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        header_content = tk.Frame(header_frame, bg="#1E3A5F")
        header_content.pack(expand=True, fill=tk.BOTH, padx=30)

        title_label = tk.Label(
            header_content,
            text="مسؤولي مواد البرامج التدريبية",
            font=("Tajawal", 24, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack(side=tk.LEFT, pady=20)

        # إطار الأزرار في الشريط العلوي
        header_buttons = tk.Frame(header_content, bg="#1E3A5F")
        header_buttons.pack(side=tk.RIGHT, pady=20)

        # زر تصدير البيانات
        export_btn = tk.Button(
            header_buttons,
            text="تصدير بيانات مسؤولي المواد",
            font=("Tajawal", 13, "bold"),
            bg="#FF9800",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=lambda: self._export_subject_teachers_data()
        )
        export_btn.pack(side=tk.LEFT)

        # إطار البحث
        search_frame = tk.Frame(subjects_window, bg=self.COLORS["surface"], height=70)
        search_frame.pack(fill=tk.X, padx=15, pady=(0, 10))
        search_frame.pack_propagate(False)

        search_container = tk.Frame(search_frame, bg=self.COLORS["surface"])
        search_container.pack(side=tk.LEFT, padx=20, pady=20)

        tk.Label(
            search_container,
            text="البحث عن دورة:",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        search_entry = tk.Entry(
            search_container,
            font=("Tajawal", 13),
            width=30,
            bd=2,
            relief=tk.FLAT,
            highlightthickness=2,
            highlightcolor="#1E3A5F"
        )
        search_entry.pack(side=tk.LEFT)

        # خط فاصل
        separator = tk.Frame(subjects_window, bg=self.COLORS["border"], height=2)
        separator.pack(fill=tk.X, padx=15, pady=(0, 10))

        # إطار الجدول الرئيسي
        main_table_frame = tk.Frame(subjects_window, bg=self.COLORS["table_bg"], bd=2, relief=tk.RIDGE)
        main_table_frame.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 10))

        table_frame = tk.Frame(main_table_frame, bg=self.COLORS["table_bg"])
        table_frame.pack(fill=tk.BOTH, expand=True, padx=3, pady=3)

        # شريط التمرير
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical")
        v_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # تنسيق Treeview
        style = ttk.Style()
        style.configure("Subjects.Treeview",
                        background="#FFFFFF",
                        foreground="#000000",
                        rowheight=45,
                        fieldbackground="#FFFFFF",
                        font=("Tajawal", 14, "normal"),
                        borderwidth=1,
                        relief="solid")

        style.configure("Subjects.Treeview.Heading",
                        font=("Tajawal", 16, "bold"),
                        background="#1E3A5F",
                        foreground="#FFFFFF",
                        relief="raised",
                        borderwidth=1,
                        padding=[10, 8])

        # إنشاء الجدول
        subjects_tree = ttk.Treeview(
            table_frame,
            columns=("course_name", "subject_name", "responsible", "members", "total"),
            show="tree headings",
            style="Subjects.Treeview",
            yscrollcommand=v_scrollbar.set,
            height=12
        )

        # إخفاء عمود الشجرة
        subjects_tree.column("#0", width=0, stretch=tk.NO)

        # تكوين الأعمدة
        column_configs = [
            ("course_name", "اسم الدورة", 300, tk.CENTER),
            ("subject_name", "اسم المادة", 250, tk.CENTER),
            ("responsible", "رئيس المادة", 200, tk.CENTER),
            ("members", "الأعضاء", 300, tk.CENTER),
            ("total", "العدد", 100, tk.CENTER)
        ]

        for col_id, heading, width, anchor in column_configs:
            subjects_tree.column(col_id, width=width, anchor=anchor, minwidth=width - 30)
            subjects_tree.heading(col_id, text=heading, anchor=tk.CENTER)

        # تكوين ألوان الصفوف
        subjects_tree.tag_configure('oddrow', background='#FFFFFF')
        subjects_tree.tag_configure('evenrow', background='#F0F8FF')

        subjects_tree.pack(fill=tk.BOTH, expand=True)
        v_scrollbar.config(command=subjects_tree.yview)

        # دالة تحميل البيانات
        def load_subject_teachers(search_term=""):
            # مسح الجدول
            for item in subjects_tree.get_children():
                subjects_tree.delete(item)

            try:
                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات والمواد
                if search_term:
                    cursor.execute("""
                        SELECT DISTINCT cn.name, cds.subject_name
                        FROM course_names cn
                        INNER JOIN course_default_subjects cds ON cn.id = cds.course_name_id
                        WHERE cn.is_active = 1 AND LOWER(cn.name) LIKE LOWER(?)
                        ORDER BY cn.name, cds.subject_order
                    """, (f'%{search_term}%',))
                else:
                    cursor.execute("""
                        SELECT DISTINCT cn.name, cds.subject_name
                        FROM course_names cn
                        INNER JOIN course_default_subjects cds ON cn.id = cds.course_name_id
                        WHERE cn.is_active = 1
                        ORDER BY cn.name, cds.subject_order
                    """)

                course_subjects = cursor.fetchall()

                for index, (course_name, subject_name) in enumerate(course_subjects):
                    # الحصول على رئيس المادة
                    cursor.execute("""
                        SELECT t.name, t.rank
                        FROM subject_teachers st
                        JOIN teachers t ON st.teacher_id = t.id
                        WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 1
                    """, (course_name, subject_name))

                    responsible = cursor.fetchone()
                    responsible_name = f"{responsible[1]} - {responsible[0]}" if responsible else "غير محدد"

                    # الحصول على الأعضاء
                    cursor.execute("""
                        SELECT t.name, t.rank, t.category
                        FROM subject_teachers st
                        JOIN teachers t ON st.teacher_id = t.id
                        WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                        ORDER BY t.category, t.name
                    """, (course_name, subject_name))

                    members = cursor.fetchall()

                    # تجميع الأعضاء حسب الفئة
                    city_staff = []
                    collaborators = []

                    for member in members:
                        if member[2] == "منسوبي المدينة":
                            city_staff.append(f"{member[1]} - {member[0]}")
                        else:
                            collaborators.append(f"{member[1]} - {member[0]}")

                    members_text = ""
                    if city_staff:
                        members_text += f"منسوبي المدينة ({len(city_staff)}): " + ", ".join(city_staff[:3])
                        if len(city_staff) > 3:
                            members_text += "..."

                    if collaborators:
                        if members_text:
                            members_text += " | "
                        members_text += f"متعاونين ({len(collaborators)}): " + ", ".join(collaborators[:3])
                        if len(collaborators) > 3:
                            members_text += "..."

                    if not members_text:
                        members_text = "لا يوجد أعضاء"

                    # العدد الكلي
                    total_count = len(members) + (1 if responsible else 0)

                    # تحديد التاج
                    tag = 'evenrow' if index % 2 == 0 else 'oddrow'

                    # إضافة للجدول
                    subjects_tree.insert("", tk.END, values=(
                        course_name,
                        subject_name,
                        responsible_name,
                        members_text,
                        total_count
                    ), tags=(tag,))

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ في تحميل البيانات: {str(e)}")

        # دالة إدارة مدرسي المادة
        def manage_subject_teachers(event=None):
            """إدارة مدرسي مادة معينة"""
            selection = subjects_tree.selection()
            if not selection:
                return

            item = subjects_tree.item(selection[0])
            course_name = item['values'][0]
            subject_name = item['values'][1]

            # نافذة إدارة مدرسي المادة
            manage_window = tk.Toplevel(subjects_window)
            manage_window.title(f"إدارة مدرسي المادة - {subject_name}")
            manage_window.geometry("1200x750")
            manage_window.configure(bg="#F5F5F5")
            manage_window.resizable(True, True)
            manage_window.minsize(1100, 700)

            # توسيط النافذة
            manage_window.update_idletasks()
            x = (manage_window.winfo_screenwidth() - 1200) // 2
            y = (manage_window.winfo_screenheight() - 750) // 2
            manage_window.geometry(f"1200x750+{x}+{y}")

            # شريط العنوان الرئيسي
            header_frame = tk.Frame(manage_window, bg="#1E3A5F", height=70)
            header_frame.pack(fill=tk.X)
            header_frame.pack_propagate(False)

            header_content = tk.Frame(header_frame, bg="#1E3A5F")
            header_content.pack(expand=True)

            tk.Label(
                header_content,
                text="إدارة مدرسي المادة",
                font=("Tajawal", 20, "bold"),
                bg="#1E3A5F",
                fg="white"
            ).pack()

            tk.Label(
                header_content,
                text=f"الدورة: {course_name} | المادة: {subject_name}",
                font=("Tajawal", 14),
                bg="#1E3A5F",
                fg="#E0E0E0"
            ).pack()

            # إطار المحتوى الرئيسي
            main_container = tk.Frame(manage_window, bg="#F5F5F5")
            main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=15)

            # القسم الأول: رئيس المادة
            responsible_frame = tk.Frame(main_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            responsible_frame.pack(fill=tk.X, pady=(0, 15))

            # رأس قسم المسؤول
            resp_header = tk.Frame(responsible_frame, bg="#2C3E50", height=40)
            resp_header.pack(fill=tk.X)
            resp_header.pack_propagate(False)

            tk.Label(
                resp_header,
                text=f"رئيس مادة ({subject_name})",
                font=("Tajawal", 14, "bold"),
                bg="#2C3E50",
                fg="white"
            ).pack(expand=True)

            # محتوى قسم المسؤول
            resp_content = tk.Frame(responsible_frame, bg="#FFFFFF", padx=20, pady=15)
            resp_content.pack(fill=tk.BOTH)

            # تقسيم أفقي
            resp_content.grid_columnconfigure(0, weight=1)
            resp_content.grid_columnconfigure(1, weight=1)

            # الجانب الأيمن - المسؤول الحالي
            right_resp = tk.Frame(resp_content, bg="#FFFFFF")
            right_resp.grid(row=0, column=1, sticky="nsew", padx=(10, 0))

            tk.Label(
                right_resp,
                text="الرئيس الحالي",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # إطار عرض المسؤول
            current_resp_frame = tk.Frame(right_resp, bg="#F8F9FA", relief=tk.RIDGE, bd=1, height=70)
            current_resp_frame.pack(fill=tk.X)
            current_resp_frame.pack_propagate(False)

            current_responsible_label = tk.Label(
                current_resp_frame,
                text="",
                font=("Tajawal", 12),
                bg="#F8F9FA",
                fg="#333333",
                justify=tk.CENTER
            )
            current_responsible_label.pack(expand=True)

            # زر حذف المسؤول
            remove_resp_btn = tk.Button(
                right_resp,
                text="حذف الرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_resp_btn.pack(pady=(10, 0))

            # الجانب الأيسر - البحث وتعيين
            left_resp = tk.Frame(resp_content, bg="#FFFFFF")
            left_resp.grid(row=0, column=0, sticky="nsew", padx=(0, 10))

            tk.Label(
                left_resp,
                text="تعيين رئيس جديد",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # البحث
            search_container = tk.Frame(left_resp, bg="#FFFFFF")
            search_container.pack(fill=tk.X, pady=(0, 8))

            tk.Label(
                search_container,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            responsible_search_entry = tk.Entry(
                search_container,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1E3A5F"
            )
            responsible_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث
            list_frame = tk.Frame(left_resp, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame, width=12)
            scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            responsible_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 11),
                height=4,
                bd=0,
                highlightthickness=0,
                yscrollcommand=scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#E3F2FD",
                selectforeground="#0D47A1"
            )
            responsible_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=responsible_listbox.yview)

            # زر التعيين
            assign_resp_btn = tk.Button(
                left_resp,
                text="تعيين كرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#3498DB",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            assign_resp_btn.pack(pady=(8, 0))

            # القسم الثاني: أعضاء المادة - مقسم إلى قسمين
            members_container = tk.Frame(main_container, bg="#F5F5F5")
            members_container.pack(fill=tk.BOTH, expand=True)

            # القسم الأيمن - منسوبي المدينة
            city_staff_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            city_staff_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(5, 0))

            # رأس قسم منسوبي المدينة
            city_header = tk.Frame(city_staff_frame, bg="#1B4F72", height=40)
            city_header.pack(fill=tk.X)
            city_header.pack_propagate(False)

            city_header_content = tk.Frame(city_header, bg="#1B4F72")
            city_header_content.pack(expand=True)

            tk.Label(
                city_header_content,
                text="أعضاء المادة - منسوبي المدينة",
                font=("Tajawal", 14, "bold"),
                bg="#1B4F72",
                fg="white"
            ).pack(side=tk.RIGHT)

            city_count_label = tk.Label(
                city_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#1B4F72",
                fg="white"
            )
            city_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم منسوبي المدينة
            city_content = tk.Frame(city_staff_frame, bg="#FFFFFF", padx=15, pady=15)
            city_content.pack(fill=tk.BOTH, expand=True)

            # البحث في منسوبي المدينة
            city_search_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                city_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            city_search_entry = tk.Entry(
                city_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1B4F72"
            )
            city_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث لمنسوبي المدينة
            city_search_list_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            city_search_scrollbar = tk.Scrollbar(city_search_list_frame, width=12)
            city_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_search_listbox = tk.Listbox(
                city_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D4E6F1",
                selectforeground="#1B4F72"
            )
            city_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_search_scrollbar.config(command=city_search_listbox.yview)

            # إطار الأزرار لمنسوبي المدينة
            city_buttons_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_buttons_frame.pack(pady=(5, 10))

            # زر إضافة منسوب مدينة
            add_city_btn = tk.Button(
                city_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#2874A6",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_city_btn.pack(side=tk.RIGHT, padx=(0, 10))

            # زر حذف منسوب مدينة
            delete_city_btn = tk.Button(
                city_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            delete_city_btn.pack(side=tk.RIGHT)

            # خط فاصل
            tk.Frame(city_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من منسوبي المدينة
            tk.Label(
                city_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#1B4F72"
            ).pack(anchor=tk.E, pady=(0, 10))

            city_members_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_members_frame.pack(fill=tk.BOTH, expand=True)

            city_members_scrollbar = tk.Scrollbar(city_members_frame, width=12)
            city_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_members_listbox = tk.Listbox(
                city_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_members_scrollbar.set,
                bg="#EBF5FB",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            city_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_members_scrollbar.config(command=city_members_listbox.yview)

            # القسم الأيسر - المتعاونين
            collaborators_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            collaborators_frame.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 5))

            # رأس قسم المتعاونين
            collab_header = tk.Frame(collaborators_frame, bg="#424949", height=40)
            collab_header.pack(fill=tk.X)
            collab_header.pack_propagate(False)

            collab_header_content = tk.Frame(collab_header, bg="#424949")
            collab_header_content.pack(expand=True)

            tk.Label(
                collab_header_content,
                text="أعضاء المادة - المتعاونين",
                font=("Tajawal", 14, "bold"),
                bg="#424949",
                fg="white"
            ).pack(side=tk.RIGHT)

            collab_count_label = tk.Label(
                collab_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#424949",
                fg="white"
            )
            collab_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم المتعاونين
            collab_content = tk.Frame(collaborators_frame, bg="#FFFFFF", padx=15, pady=15)
            collab_content.pack(fill=tk.BOTH, expand=True)

            # البحث في المتعاونين
            collab_search_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                collab_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            collab_search_entry = tk.Entry(
                collab_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#424949"
            )
            collab_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث للمتعاونين
            collab_search_list_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            collab_search_scrollbar = tk.Scrollbar(collab_search_list_frame, width=12)
            collab_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_search_listbox = tk.Listbox(
                collab_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D5DBDB",
                selectforeground="#212F3C"
            )
            collab_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_search_scrollbar.config(command=collab_search_listbox.yview)

            # إطار الأزرار للمتعاونين
            collab_buttons_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_buttons_frame.pack(pady=(5, 10))

            # زر إضافة متعاون
            add_collab_btn = tk.Button(
                collab_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#5D6D7E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_collab_btn.pack(side=tk.RIGHT, padx=(0, 10))

            # زر حذف متعاون
            delete_collab_btn = tk.Button(
                collab_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            delete_collab_btn.pack(side=tk.RIGHT)

            # خط فاصل
            tk.Frame(collab_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من المتعاونين
            tk.Label(
                collab_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#424949"
            ).pack(anchor=tk.E, pady=(0, 10))

            collab_members_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_members_frame.pack(fill=tk.BOTH, expand=True)

            collab_members_scrollbar = tk.Scrollbar(collab_members_frame, width=12)
            collab_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_members_listbox = tk.Listbox(
                collab_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_members_scrollbar.set,
                bg="#F4F6F6",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            collab_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_members_scrollbar.config(command=collab_members_listbox.yview)

            # الشريط السفلي
            footer_frame = tk.Frame(manage_window, bg="#1E3A5F", height=50)
            footer_frame.pack(fill=tk.X)
            footer_frame.pack_propagate(False)

            close_btn = tk.Button(
                footer_frame,
                text="إغلاق",
                font=("Tajawal", 13, "bold"),
                bg="#34495E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=35,
                pady=10,
                command=lambda: [manage_window.destroy(), load_subject_teachers(search_entry.get())]
            )
            close_btn.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            # maps لتخزين معرفات المدرسين
            manage_window.city_members_map = {}
            manage_window.collab_members_map = {}
            manage_window.city_search_map = {}
            manage_window.collab_search_map = {}
            manage_window.responsible_map = {}

            # تحميل البيانات الحالية
            def load_current_data():
                cursor = self.db_conn.cursor()

                # تحميل رئيس المادة
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 1
                """, (course_name, subject_name))

                responsible = cursor.fetchone()
                if responsible:
                    current_responsible_label.config(
                        text=f"{responsible[1]}\n{responsible[2]} - {responsible[3]}",
                        font=("Tajawal", 11, "bold")
                    )
                    remove_resp_btn.config(state=tk.NORMAL)
                else:
                    current_responsible_label.config(
                        text="لم يتم تعيين رئيس بعد",
                        fg="#999999",
                        font=("Tajawal", 11, "italic")
                    )
                    remove_resp_btn.config(state=tk.DISABLED)

                # تحميل أعضاء منسوبي المدينة
                city_members_listbox.delete(0, tk.END)
                manage_window.city_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (course_name, subject_name))

                city_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace) in enumerate(city_members):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_members_listbox.insert(tk.END, display_text)
                    manage_window.city_members_map[i] = teacher_id

                city_count_label.config(text=f"العدد: {len(city_members)}")

                # تحميل أعضاء المتعاونين
                collab_members_listbox.delete(0, tk.END)
                manage_window.collab_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (course_name, subject_name))

                collab_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(collab_members):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_members_listbox.insert(tk.END, display_text)
                    manage_window.collab_members_map[i] = teacher_id

                collab_count_label.config(text=f"العدد: {len(collab_members)}")

            # البحث عن المدرسين للمسؤول
            def search_responsible_teachers(*args):
                search_term = responsible_search_entry.get().strip()
                responsible_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT id, name, rank, workplace, category
                    FROM teachers 
                    WHERE LOWER(name) LIKE LOWER(?)
                    AND id NOT IN (
                        SELECT teacher_id FROM subject_teachers 
                        WHERE course_name = ? AND subject_name = ?
                    )
                    ORDER BY name
                    LIMIT 20
                """, (f'%{search_term}%', course_name, subject_name))

                teachers = cursor.fetchall()
                manage_window.responsible_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    if category:
                        display_text += f" [{category}]"
                    responsible_listbox.insert(tk.END, display_text)
                    manage_window.responsible_map[i] = teacher_id

            # البحث في منسوبي المدينة
            def search_city_teachers(*args):
                search_term = city_search_entry.get().strip()
                city_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category = 'منسوبي المدينة'
                    AND t.id NOT IN (
                        SELECT teacher_id FROM subject_teachers 
                        WHERE course_name = ? AND subject_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name, subject_name))

                teachers = cursor.fetchall()
                manage_window.city_search_map.clear()
                for i, (teacher_id, name, rank, workplace) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_search_listbox.insert(tk.END, display_text)
                    manage_window.city_search_map[i] = teacher_id

            # البحث في المتعاونين
            def search_collab_teachers(*args):
                search_term = collab_search_entry.get().strip()
                collab_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    AND t.id NOT IN (
                        SELECT teacher_id FROM subject_teachers 
                        WHERE course_name = ? AND subject_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name, subject_name))

                teachers = cursor.fetchall()
                manage_window.collab_search_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_search_listbox.insert(tk.END, display_text)
                    manage_window.collab_search_map[i] = teacher_id

            # تعيين رئيس المادة
            def assign_responsible():
                selection = responsible_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.responsible_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد تعيين هذا المدرس كرئيس للمادة؟"):
                    try:
                        cursor = self.db_conn.cursor()

                        # إلغاء أي مسؤول سابق
                        cursor.execute("""
                            UPDATE subject_teachers 
                            SET is_responsible = 0 
                            WHERE course_name = ? AND subject_name = ? AND is_responsible = 1
                        """, (course_name, subject_name))

                        # تعيين المسؤول الجديد
                        cursor.execute("""
                            INSERT INTO subject_teachers 
                            (course_name, subject_name, teacher_id, is_responsible, created_date)
                            VALUES (?, ?, ?, 1, ?)
                            ON CONFLICT(course_name, subject_name, teacher_id) 
                            DO UPDATE SET is_responsible = 1
                        """, (course_name, subject_name, teacher_id,
                              datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                        self.db_conn.commit()

                        load_current_data()
                        responsible_search_entry.delete(0, tk.END)
                        responsible_listbox.delete(0, tk.END)

                        messagebox.showinfo("نجاح", "تم تعيين رئيس المادة بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف رئيس المادة
            def remove_responsible():
                if messagebox.askyesno("تأكيد", "هل تريد حذف رئيس المادة الحالي؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM subject_teachers 
                            WHERE course_name = ? AND subject_name = ? AND is_responsible = 1
                        """, (course_name, subject_name))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف رئيس المادة بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة منسوب مدينة
            def add_city_member():
                selection = city_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.city_search_map[selection[0]]

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO subject_teachers 
                        (course_name, subject_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, ?, 0, ?)
                    """, (course_name, subject_name, teacher_id,
                          datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    city_search_entry.delete(0, tk.END)
                    city_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة متعاون
            def add_collab_member():
                selection = collab_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.collab_search_map[selection[0]]

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO subject_teachers 
                        (course_name, subject_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, ?, 0, ?)
                    """, (course_name, subject_name, teacher_id,
                          datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    collab_search_entry.delete(0, tk.END)
                    collab_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف منسوب مدينة
            def remove_city_member():
                selection = city_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو من قائمة الأعضاء الحاليين للحذف")
                    return

                teacher_id = manage_window.city_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من المادة؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM subject_teachers 
                            WHERE course_name = ? AND subject_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, subject_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف العضو بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف متعاون
            def remove_collab_member():
                selection = collab_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو من قائمة الأعضاء الحاليين للحذف")
                    return

                teacher_id = manage_window.collab_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من المادة؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM subject_teachers 
                            WHERE course_name = ? AND subject_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, subject_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف العضو بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # ربط الأحداث
            responsible_search_entry.bind('<KeyRelease>', search_responsible_teachers)
            city_search_entry.bind('<KeyRelease>', search_city_teachers)
            collab_search_entry.bind('<KeyRelease>', search_collab_teachers)

            assign_resp_btn.config(command=assign_responsible)
            remove_resp_btn.config(command=remove_responsible)
            add_city_btn.config(command=add_city_member)
            add_collab_btn.config(command=add_collab_member)
            delete_city_btn.config(command=remove_city_member)
            delete_collab_btn.config(command=remove_collab_member)

            # تحميل البيانات الأولية
            load_current_data()

        # ربط النقر على الجدول
        subjects_tree.bind("<Double-Button-1>", manage_subject_teachers)

        # البحث الديناميكي
        def dynamic_search(*args):
            search_term = search_entry.get().strip()
            load_subject_teachers(search_term)

        # ربط البحث الديناميكي
        search_entry.bind('<KeyRelease>', dynamic_search)

        # تسمية للمساعدة
        help_label = tk.Label(
            subjects_window,
            text="💡 انقر مرتين على أي مادة لإدارة مدرسيها",
            font=("Tajawal", 12),
            bg=self.COLORS["background"],
            fg="#666"
        )
        help_label.pack(pady=5)

        # زر الإغلاق
        close_btn = tk.Button(
            subjects_window,
            text="إغلاق",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["dark"],
            fg="white",
            bd=0,
            padx=30,
            pady=12,
            cursor="hand2",
            command=subjects_window.destroy
        )
        close_btn.pack(pady=10)

        # تحميل البيانات
        load_subject_teachers()

    def _export_subject_teachers_data(self):
        """تصدير بيانات مسؤولي المواد إلى Excel"""
        if not EXCEL_AVAILABLE:
            messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
            return

        try:
            from tkinter import filedialog
            import pandas as pd
            from openpyxl import Workbook
            from openpyxl.styles import Alignment, Font, PatternFill, Border, Side
            from openpyxl.utils import get_column_letter

            # اختيار مكان الحفظ
            file_path = filedialog.asksaveasfilename(
                title="حفظ بيانات مسؤولي المواد",
                defaultextension=".xlsx",
                initialfile=f"مسؤولي_المواد_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
            )

            if not file_path:
                return

            cursor = self.db_conn.cursor()

            # إنشاء ملف Excel
            wb = Workbook()
            ws = wb.active
            ws.title = "مسؤولي المواد"

            # تعيين اتجاه الورقة من اليمين لليسار
            ws.sheet_view.rightToLeft = True

            # العناوين
            headers = ["م", "اسم الدورة", "اسم المادة", "رئيس المادة", "أعضاء منسوبي المدينة", "أعضاء متعاونين",
                       "العدد الكلي"]

            # كتابة العناوين
            for col, header in enumerate(headers, 1):
                cell = ws.cell(row=1, column=col, value=header)
                cell.font = Font(name='Arial', size=14, bold=True, color="FFFFFF")
                cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
                cell.fill = PatternFill(start_color="9C27B0", end_color="9C27B0", fill_type="solid")

            # تنسيق الحدود
            thin_border = Border(
                left=Side(style='thin'),
                right=Side(style='thin'),
                top=Side(style='thin'),
                bottom=Side(style='thin')
            )

            # الحصول على جميع الدورات والمواد
            cursor.execute("""
                SELECT DISTINCT cn.name, cds.subject_name
                FROM course_names cn
                INNER JOIN course_default_subjects cds ON cn.id = cds.course_name_id
                WHERE cn.is_active = 1
                ORDER BY cn.name, cds.subject_order
            """)

            course_subjects = cursor.fetchall()
            row_num = 2

            for idx, (course_name, subject_name) in enumerate(course_subjects, 1):
                # رئيس المادة
                cursor.execute("""
                    SELECT t.name, t.rank
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 1
                """, (course_name, subject_name))
                responsible = cursor.fetchone()
                responsible_text = f"{responsible[0]} - {responsible[1]}" if responsible else "غير محدد"

                # أعضاء منسوبي المدينة
                cursor.execute("""
                    SELECT t.name, t.rank
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (course_name, subject_name))
                city_members = cursor.fetchall()
                city_text = "\n".join([f"{m[0]} - {m[1]}" for m in city_members]) if city_members else "لا يوجد"

                # أعضاء متعاونين
                cursor.execute("""
                    SELECT t.name, t.rank, t.category
                    FROM subject_teachers st
                    JOIN teachers t ON st.teacher_id = t.id
                    WHERE st.course_name = ? AND st.subject_name = ? AND st.is_responsible = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (course_name, subject_name))
                collab_members = cursor.fetchall()
                collab_text = "\n".join(
                    [f"{c[0]} - {c[1]} ({c[2]})" for c in collab_members]) if collab_members else "لا يوجد"

                # العدد الكلي
                total = len(city_members) + len(collab_members) + (1 if responsible else 0)

                # كتابة الصف
                data_row = [idx, course_name, subject_name, responsible_text, city_text, collab_text, total]
                for col, value in enumerate(data_row, 1):
                    cell = ws.cell(row=row_num, column=col, value=value)
                    cell.font = Font(name='Arial', size=12)
                    cell.alignment = Alignment(
                        horizontal='right' if col > 1 else 'center',
                        vertical='center',
                        wrap_text=True
                    )
                    cell.border = thin_border

                    # تلوين الصفوف بالتناوب
                    if row_num % 2 == 0:
                        cell.fill = PatternFill(start_color="F3E5F5", end_color="F3E5F5", fill_type="solid")

                row_num += 1

            # تعيين عرض الأعمدة
            ws.column_dimensions['A'].width = 8  # م
            ws.column_dimensions['B'].width = 35  # اسم الدورة
            ws.column_dimensions['C'].width = 35  # اسم المادة
            ws.column_dimensions['D'].width = 30  # رئيس المادة
            ws.column_dimensions['E'].width = 45  # أعضاء منسوبي المدينة
            ws.column_dimensions['F'].width = 45  # أعضاء متعاونين
            ws.column_dimensions['G'].width = 12  # العدد الكلي

            # إضافة ورقة الملخص
            summary_ws = wb.create_sheet("الملخص")
            summary_ws.sheet_view.rightToLeft = True

            # عنوان الملخص
            summary_ws['A1'] = "ملخص بيانات مسؤولي المواد"
            summary_ws['A1'].font = Font(name='Arial', size=16, bold=True)
            summary_ws.merge_cells('A1:B1')

            # حساب الإحصائيات
            cursor.execute("""
                SELECT COUNT(DISTINCT course_name || '-' || subject_name) 
                FROM subject_teachers
            """)
            total_subjects_with_teachers = cursor.fetchone()[0]

            cursor.execute("""
                SELECT COUNT(DISTINCT teacher_id) 
                FROM subject_teachers 
                WHERE is_responsible = 1
            """)
            total_responsible = cursor.fetchone()[0]

            cursor.execute("""
                SELECT COUNT(DISTINCT teacher_id) 
                FROM subject_teachers st
                JOIN teachers t ON st.teacher_id = t.id
                WHERE st.is_responsible = 0 AND t.category = 'منسوبي المدينة'
            """)
            total_city_members = cursor.fetchone()[0]

            cursor.execute("""
                SELECT COUNT(DISTINCT teacher_id) 
                FROM subject_teachers st
                JOIN teachers t ON st.teacher_id = t.id
                WHERE st.is_responsible = 0 AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
            """)
            total_collab_members = cursor.fetchone()[0]

            # كتابة الإحصائيات
            summary_data = [
                ["إجمالي المواد", len(course_subjects)],
                ["المواد التي لها مدرسين", total_subjects_with_teachers],
                ["إجمالي رؤساء المواد", total_responsible],
                ["إجمالي أعضاء منسوبي المدينة", total_city_members],
                ["إجمالي الأعضاء المتعاونين", total_collab_members],
                ["", ""],
                ["تاريخ التصدير", datetime.now().strftime('%Y-%m-%d')],
                ["وقت التصدير", datetime.now().strftime('%H:%M:%S')]
            ]

            for idx, (label, value) in enumerate(summary_data, 3):
                summary_ws[f'A{idx}'] = label
                summary_ws[f'B{idx}'] = value
                summary_ws[f'A{idx}'].font = Font(name='Arial', size=12, bold=True)
                summary_ws[f'B{idx}'].font = Font(name='Arial', size=12)

            # تعيين عرض الأعمدة للملخص
            summary_ws.column_dimensions['A'].width = 30
            summary_ws.column_dimensions['B'].width = 20

            # حفظ الملف
            wb.save(file_path)
            messagebox.showinfo("نجاح", f"تم تصدير بيانات مسؤولي المواد بنجاح\n{file_path}")

        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

    def _search_teachers(self):
        """البحث الديناميكي في المدرسين"""
        # الحصول على نص البحث
        search_term = self.search_var.get().strip()
        filter_category = self.filter_var.get()

        # مسح جميع العناصر
        for item in self.teachers_tree.get_children():
            self.teachers_tree.delete(item)

        try:
            cursor = self.db_conn.cursor()

            # بناء استعلام SQL
            if filter_category == "الكل":
                if search_term:
                    # البحث في الاسم ورقم الهوية
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE name LIKE ? OR id_number LIKE ?
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    # عرض الكل
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        ORDER BY name
                    """)
            elif filter_category == "منسوبي المدينة":
                if search_term:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category = 'منسوبي المدينة' 
                        AND (name LIKE ? OR id_number LIKE ?)
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category = 'منسوبي المدينة'
                        ORDER BY name
                    """)
            elif filter_category == "متعاونين":
                if search_term:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category IN ('متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد') 
                        AND (name LIKE ? OR id_number LIKE ?)
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category IN ('متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY name
                    """)

            teachers = cursor.fetchall()

            # إضافة النتائج إلى الشجرة
            for index, teacher in enumerate(teachers):
                display_data = (
                    teacher[1],  # name
                    teacher[2],  # rank
                    teacher[3],  # id_number
                    teacher[4],  # workplace
                    teacher[5],  # qualification
                    teacher[6]  # phone
                )

                tag = 'evenrow' if index % 2 == 0 else 'oddrow'
                item = self.teachers_tree.insert("", tk.END, values=display_data, tags=(tag, f"id_{teacher[0]}"))

            # تحديث العدادات
            self._update_teachers_count()

            # تحديث نص زر التصدير
            if filter_category != "الكل":
                self.export_btn.config(text=f"📊 تصدير بيانات {filter_category}")
            else:
                self.export_btn.config(text="📊 تصدير جميع البيانات")

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في البحث: {str(e)}")
            print(f"تفاصيل الخطأ: {str(e)}")  # للتشخيص

    def _filter_teachers(self):
        """تصفية المدرسين حسب الفئة"""
        self._search_teachers()  # استخدام نفس دالة البحث

    def _update_teachers_count(self):
        """تحديث عدد المدرسين مع إظهار التفاصيل"""
        try:
            cursor = self.db_conn.cursor()

            # عدد منسوبي المدينة
            cursor.execute("SELECT COUNT(*) FROM teachers WHERE category = 'منسوبي المدينة'")
            city_count = cursor.fetchone()[0]

            # عدد المتعاونين (جميع الفئات بما فيها "متعاون" فقط)
            cursor.execute("""
                SELECT COUNT(*) FROM teachers 
                WHERE category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
            """)
            collaborator_count = cursor.fetchone()[0]

            # العدد الإجمالي
            total_count = city_count + collaborator_count

            # تحديث النص
            text = f"إجمالي المدرسين: {total_count} | منسوبي المدينة: {city_count} | المتعاونين: {collaborator_count}"

            self.teacher_count_label.config(text=text)

        except Exception as e:
            print(f"خطأ في تحديث العدادات: {e}")

        finally:
            cursor.close()

    def _export_filtered_data(self):
        """تصدير البيانات المفلترة إلى Excel"""
        if not EXCEL_AVAILABLE:
            messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
            return

        try:
            from tkinter import filedialog

            # الحصول على البيانات المعروضة حالياً
            data = []
            for item in self.teachers_tree.get_children():
                values = self.teachers_tree.item(item)['values']
                data.append(values)

            if not data:
                messagebox.showinfo("تنبيه", "لا توجد بيانات للتصدير")
                return

            # تحديد اسم الملف
            filter_category = self.filter_combo.get() if hasattr(self, 'filter_combo') else "الكل"
            default_name = f"مدرسين_{filter_category}" if filter_category != "الكل" else "جميع_المدرسين"

            file_path = filedialog.asksaveasfilename(
                title="حفظ ملف Excel",
                defaultextension=".xlsx",
                initialfile=f"{default_name}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
            )

            if not file_path:
                return

            # إنشاء DataFrame
            columns = ["الاسم", "الرتبة", "رقم الهوية", "جهة العمل", "المؤهل الدراسي", "رقم الجوال"]
            df = pd.DataFrame(data, columns=columns)

            # إنشاء ملف Excel مع التنسيق
            with pd.ExcelWriter(file_path, engine='openpyxl') as writer:
                # كتابة البيانات
                df.to_excel(writer, sheet_name='المدرسون', index=False)

                # الحصول على ورقة العمل
                worksheet = writer.sheets['المدرسون']

                # تعيين اتجاه الورقة من اليمين لليسار
                worksheet.sheet_view.rightToLeft = True

                # تنسيق بسيط
                thin_border = Border(
                    left=Side(style='thin'),
                    right=Side(style='thin'),
                    top=Side(style='thin'),
                    bottom=Side(style='thin')
                )

                # تنسيق رؤوس الأعمدة
                for col_num, column in enumerate(columns, 1):
                    cell = worksheet.cell(row=1, column=col_num)
                    cell.font = Font(name='Arial', size=12, bold=True)
                    cell.alignment = Alignment(
                        horizontal='center',
                        vertical='center',
                        wrap_text=True
                    )
                    cell.border = thin_border

                # تنسيق البيانات
                for row_num in range(2, len(df) + 2):
                    for col_num in range(1, len(columns) + 1):
                        cell = worksheet.cell(row=row_num, column=col_num)
                        cell.font = Font(name='Arial', size=11)
                        cell.alignment = Alignment(
                            horizontal='right' if col_num != 3 else 'center',
                            vertical='center',
                            wrap_text=True
                        )
                        cell.border = thin_border

                # تعيين عرض الأعمدة
                column_widths = {
                    'A': 35,  # الاسم
                    'B': 20,  # الرتبة
                    'C': 15,  # رقم الهوية
                    'D': 35,  # جهة العمل
                    'E': 25,  # المؤهل
                    'F': 15  # رقم الجوال
                }

                for column, width in column_widths.items():
                    worksheet.column_dimensions[column].width = width

                # إضافة ورقة الملخص
                summary_data = {
                    'الفئة': [filter_category if filter_category != "الكل" else "جميع الفئات"],
                    'عدد المدرسين': [len(data)],
                    'تاريخ التصدير': [datetime.now().strftime('%Y-%m-%d')],
                    'وقت التصدير': [datetime.now().strftime('%H:%M:%S')],
                    'المستخدم': [self.current_user['full_name'] if self.current_user else 'غير محدد']
                }

                summary_df = pd.DataFrame(summary_data)
                summary_df.to_excel(writer, sheet_name='ملخص', index=False)

            messagebox.showinfo("نجاح", f"تم تصدير البيانات بنجاح\n{file_path}")

        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

    def _load_teachers(self):
        """تحميل بيانات المدرسين من قاعدة البيانات مع التنسيق المحسن"""
        # إعادة تعيين البحث والفلتر
        if hasattr(self, 'search_var'):
            self.search_var.set("")
        if hasattr(self, 'filter_var'):
            self.filter_var.set("الكل")

        # مسح البيانات الحالية
        for item in self.teachers_tree.get_children():
            self.teachers_tree.delete(item)

        try:
            cursor = self.db_conn.cursor()
            cursor.execute("""
                SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                FROM teachers 
                ORDER BY name
            """)

            teachers = cursor.fetchall()

            # إضافة البيانات مع ألوان متناوبة
            for index, teacher in enumerate(teachers):
                # البيانات المعروضة (بدون ID)
                display_data = (
                    teacher[1],  # name
                    teacher[2],  # rank
                    teacher[3],  # id_number
                    teacher[4],  # workplace
                    teacher[5],  # qualification
                    teacher[6]  # phone
                )

                tag = 'evenrow' if index % 2 == 0 else 'oddrow'
                item = self.teachers_tree.insert("", tk.END, values=display_data, tags=(tag, f"id_{teacher[0]}"))

            # تحديث عدادات المدرسين
            self._update_teachers_count()

            # تحديث نص زر التصدير
            if hasattr(self, 'export_btn'):
                self.export_btn.config(text="📊 تصدير جميع البيانات")

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تحميل البيانات: {str(e)}")

    def _check_categories(self):
        """التحقق من الفئات الموجودة في قاعدة البيانات"""
        try:
            cursor = self.db_conn.cursor()
            cursor.execute("SELECT DISTINCT category FROM teachers WHERE category IS NOT NULL")
            categories = cursor.fetchall()

            print("الفئات الموجودة في قاعدة البيانات:")
            for cat in categories:
                print(f"- '{cat[0]}'")

            # يمكنك استدعاء هذه الدالة مؤقتاً للتحقق من الفئات

        except Exception as e:
            print(f"خطأ: {str(e)}")

    def _fix_categories(self):
        """إصلاح أسماء الفئات في قاعدة البيانات"""
        try:
            cursor = self.db_conn.cursor()

            # تحديث الفئات لتتطابق مع القائمة المنسدلة
            updates = [
                ("متعاون", "متعاون مدني"),  # إذا كانت مخزنة كـ "متعاون" فقط
                ("متعاون عسكري متقاعد", "متعاون عسكري متقاعد"),  # للتأكد من عدم وجود مسافات زائدة
            ]

            for old_name, new_name in updates:
                cursor.execute("UPDATE teachers SET category = ? WHERE category = ? OR category LIKE ?",
                               (new_name, old_name, f"%{old_name}%"))

            self.db_conn.commit()
            messagebox.showinfo("نجاح", "تم تحديث الفئات")
            self._load_teachers()

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تحديث الفئات: {str(e)}")

    # 1. دالة إضافة مدرس جديد كاملة
    def _open_add_teacher_window(self):
        """فتح نافذة إضافة مدرس جديد مع مسار التخصص بدلاً من الدورات"""
        add_window = tk.Toplevel(self)
        add_window.title("إضافة مدرس جديد")
        add_window.geometry("700x750")
        add_window.configure(bg=self.COLORS["background"])
        add_window.transient(self)
        add_window.grab_set()

        # توسيط النافذة
        add_window.update_idletasks()
        x = (add_window.winfo_screenwidth() - 700) // 2
        y = (add_window.winfo_screenheight() - 750) // 2
        add_window.geometry(f"700x750+{x}+{y}")

        # شريط العنوان
        header_frame = tk.Frame(add_window, bg="#1E3A5F", height=80)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        title_label = tk.Label(
            header_frame,
            text="إضافة مدرس جديد",
            font=("Tajawal", 24, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack(expand=True)

        # إطار المحتوى الرئيسي
        main_frame = tk.Frame(add_window, bg=self.COLORS["background"])
        main_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

        # إطار الحقول
        form_frame = tk.Frame(main_frame, bg=self.COLORS["card"], bd=2, relief=tk.RIDGE)
        form_frame.pack(fill=tk.BOTH, expand=True)

        inner_form = tk.Frame(form_frame, bg=self.COLORS["card"], padx=40, pady=30)
        inner_form.pack(fill=tk.BOTH, expand=True)

        # تنسيق الحقول
        label_font = ("Tajawal", 14, "bold")
        entry_font = ("Tajawal", 13)

        # حقل الاسم
        tk.Label(inner_form, text="الاسم الكامل *", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=0, column=0, sticky=tk.W, pady=15)
        name_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                              highlightthickness=2, highlightcolor="#1E3A5F")
        name_entry.grid(row=0, column=1, pady=15, padx=10)

        # حقل الرتبة
        tk.Label(inner_form, text="الرتبة *", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=1, column=0, sticky=tk.W, pady=15)
        rank_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                              highlightthickness=2, highlightcolor="#1E3A5F")
        rank_entry.grid(row=1, column=1, pady=15, padx=10)

        # حقل رقم الهوية
        tk.Label(inner_form, text="رقم الهوية * (10 أرقام)", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=2, column=0, sticky=tk.W, pady=15)
        id_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                            highlightthickness=2, highlightcolor="#1E3A5F")
        id_entry.grid(row=2, column=1, pady=15, padx=10)

        # التحقق من إدخال الأرقام فقط
        def validate_id(char):
            return char.isdigit() or char == ""

        vcmd = (add_window.register(validate_id), '%S')
        id_entry.config(validate='key', validatecommand=vcmd)

        # حقل رقم الجوال
        tk.Label(inner_form, text="رقم الجوال (اختياري)", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=3, column=0, sticky=tk.W, pady=15)
        phone_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                               highlightthickness=2, highlightcolor="#1E3A5F")
        phone_entry.grid(row=3, column=1, pady=15, padx=10)

        # حقل جهة العمل
        tk.Label(inner_form, text="جهة العمل *", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=4, column=0, sticky=tk.W, pady=15)
        workplace_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                                   highlightthickness=2, highlightcolor="#1E3A5F")
        workplace_entry.grid(row=4, column=1, pady=15, padx=10)

        # حقل المؤهل الدراسي - محدث بالقيم الجديدة
        tk.Label(inner_form, text="المؤهل الدراسي *", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=5, column=0, sticky=tk.W, pady=15)

        qualifications = ["دكتوراه", "ماجستير", "دبلوم عالي", "بكالوريوس", "جامعة متوسطة", "ثانوي", "لا يوجد مؤهل"]
        qualification_var = tk.StringVar(master=add_window, value=qualifications[3])  # بكالوريوس كقيمة افتراضية

        # تنسيق Combobox
        style = ttk.Style()
        style.configure("Custom.TCombobox",
                        fieldbackground=self.COLORS["surface"],
                        borderwidth=2,
                        relief="flat",
                        font=entry_font)

        qualification_combo = ttk.Combobox(inner_form, textvariable=qualification_var,
                                           values=qualifications, font=entry_font,
                                           width=33, state="readonly", style="Custom.TCombobox")
        qualification_combo.grid(row=5, column=1, pady=15, padx=10)

        # حقل فئة المدرس
        tk.Label(inner_form, text="فئة المدرس *", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=6, column=0, sticky=tk.W, pady=15)

        categories = ["منسوبي المدينة", "متعاون مدني", "متعاون عسكري", "متعاون عسكري متقاعد"]
        category_var = tk.StringVar(master=add_window, value=categories[0])
        category_combo = ttk.Combobox(inner_form, textvariable=category_var,
                                      values=categories, font=entry_font,
                                      width=33, state="readonly", style="Custom.TCombobox")
        category_combo.grid(row=6, column=1, pady=15, padx=10)

        # حقل مسار الدورة المتخصص فيها (بدلاً من الدورات الحاصل عليها)
        tk.Label(inner_form, text="مسار الدورة المتخصص فيها", font=label_font,
                 bg=self.COLORS["card"], fg="#1E3A5F").grid(row=7, column=0, sticky=tk.W, pady=15)

        specialization_frame = tk.Frame(inner_form, bg=self.COLORS["card"])
        specialization_frame.grid(row=7, column=1, pady=15, padx=10, sticky=tk.W)

        # الحصول على مسميات الدورات من قاعدة البيانات
        cursor = self.db_conn.cursor()
        cursor.execute("SELECT id, name FROM course_names WHERE is_active = 1 ORDER BY name")
        courses = cursor.fetchall()
        course_names = ["لا يوجد"] + [course[1] for course in courses] if courses else ["لا يوجد"]

        specialization_var = tk.StringVar(master=add_window, value="لا يوجد")
        specialization_combo = ttk.Combobox(specialization_frame, textvariable=specialization_var,
                                            values=course_names, font=entry_font,
                                            width=33, state="readonly", style="Custom.TCombobox")
        specialization_combo.pack()

        # إطار الأزرار السفلي
        buttons_frame = tk.Frame(add_window, bg=self.COLORS["background"])
        buttons_frame.pack(fill=tk.X, pady=20)

        def save_teacher():
            # التحقق من البيانات المطلوبة
            name = name_entry.get().strip()
            rank = rank_entry.get().strip()
            id_number = id_entry.get().strip()
            phone = phone_entry.get().strip()
            workplace = workplace_entry.get().strip()
            qualification = qualification_var.get()
            category = category_var.get()
            specialization = specialization_var.get()

            if not all([name, rank, id_number, workplace, qualification, category]):
                messagebox.showwarning("تنبيه", "يرجى ملء جميع الحقول المطلوبة")
                return

            if len(id_number) != 10:
                messagebox.showwarning("تنبيه", "رقم الهوية يجب أن يكون 10 أرقام")
                return

            try:
                cursor = self.db_conn.cursor()

                # التحقق من عدم تكرار رقم الهوية
                cursor.execute("SELECT COUNT(*) FROM teachers WHERE id_number = ?", (id_number,))
                if cursor.fetchone()[0] > 0:
                    messagebox.showerror("خطأ", "رقم الهوية مسجل مسبقاً")
                    return

                # إدراج المدرس الجديد
                cursor.execute("""
                    INSERT INTO teachers (name, rank, id_number, phone, workplace, 
                                        qualification, category, created_date)
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                """, (name, rank, id_number, phone, workplace, qualification, category,
                      datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                teacher_id = cursor.lastrowid

                # إضافة مسار التخصص إذا كان محدداً
                if specialization != "لا يوجد":
                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (specialization, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                self.db_conn.commit()
                messagebox.showinfo("نجاح", "تم إضافة المدرس بنجاح")
                add_window.destroy()
                self._load_teachers()

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

        # زر الحفظ
        save_btn = tk.Button(
            buttons_frame,
            text="💾 حفظ البيانات",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["success"],
            fg="white",
            padx=30,
            pady=12,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=save_teacher
        )
        save_btn.pack(side=tk.LEFT, padx=20)

        # تأثيرات hover
        save_btn.bind("<Enter>", lambda e: save_btn.config(bg="#2e7d32"))
        save_btn.bind("<Leave>", lambda e: save_btn.config(bg=self.COLORS["success"]))

        # زر الإلغاء
        cancel_btn = tk.Button(
            buttons_frame,
            text="❌ إلغاء",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["danger"],
            fg="white",
            padx=30,
            pady=12,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=add_window.destroy
        )
        cancel_btn.pack(side=tk.RIGHT, padx=20)

        # تأثيرات hover
        cancel_btn.bind("<Enter>", lambda e: cancel_btn.config(bg="#c82333"))
        cancel_btn.bind("<Leave>", lambda e: cancel_btn.config(bg=self.COLORS["danger"]))
