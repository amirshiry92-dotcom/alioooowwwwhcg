import tkinter as tk
from tkinter import messagebox
import json
import os
from datetime import datetime


# ==========================================
# SETTINGS     شماره کارت:  6219  8613  5376  6466
# ==========================================

DATA_FILE = "data.json"

BG = "#f3f4f6"
NAVY = "#17033E"
DARK_NAVY = "#0b2742"
WHITE = "#ffffff"
GRAY = "#6b7280"
DARK = "#1f2937"
LIGHT_GRAY = "#e5e7eb"
GREEN = "#15803d"
RED = "#b91c1c"

ACCOUNT_CARD_NUMBER = "6219861353766466"

balance = 1000000
amount = 0
card_number = ""
transactions = []

# اطلاعات وام فعال
active_loan = None
loan_timer_id = None

# پنج گزینه وام
LOAN_OPTIONS = [
    500000,
    1000000,
    2000000,
    5000000,
    10000000
]


# ==========================================
# WINDOW
# ==========================================

window = tk.Tk()
window.title("Offline Bank")
window.geometry("430x700")
window.resizable(False, False)
window.configure(bg=BG)


# ==========================================
# DATA
# ==========================================

def save_data():
    data = {
        "balance": balance,
        "transactions": transactions,
        "active_loan": active_loan
    }

    try:
        with open(DATA_FILE, "w", encoding="utf-8") as file:
            json.dump(
                data,
                file,
                ensure_ascii=False,
                indent=4
            )
    except OSError:
        messagebox.showerror(
            "خطا",
            "ذخیره اطلاعات انجام نشد."
        )


def load_data():
    global balance
    global transactions
    global active_loan

    if not os.path.exists(DATA_FILE):
        save_data()
        return

    try:
        with open(DATA_FILE, "r", encoding="utf-8") as file:
            data = json.load(file)

        balance = data.get("balance", 1000000)
        transactions = data.get("transactions", [])
        active_loan = data.get("active_loan", None)

    except (OSError, json.JSONDecodeError):
        balance = 1000000
        transactions = []
        active_loan = None


# ==========================================
# HELPERS
# ==========================================

def clear_page():
    for widget in window.winfo_children():
        widget.destroy()


def money(value):
    return f"{value:,}"


def add_transaction(transaction_type, transaction_amount, card=""):
    now = datetime.now()

    transactions.append({
        "type": transaction_type,
        "amount": transaction_amount,
        "card": card,
        "date": now.strftime("%Y/%m/%d"),
        "time": now.strftime("%H:%M:%S"),
        "status": "موفق"
    })


# ==========================================
# MONEY FORMAT
# ==========================================

def format_money_entry(event):
    entry = event.widget

    text = entry.get()

    digits = ""

    for character in text:
        if character.isdigit():
            digits += character

    if not digits:
        return

    try:
        value = int(digits)
    except ValueError:
        return

    formatted = f"{value:,}"

    entry.delete(0, tk.END)
    entry.insert(0, formatted)
    entry.icursor(tk.END)


# ==========================================
# CARD FORMAT
# ==========================================

def format_card_entry(event):
    entry = event.widget

    text = entry.get()

    digits = ""

    for character in text:
        if character.isdigit():
            digits += character

    digits = digits[:16]

    groups = []

    for i in range(0, len(digits), 4):
        groups.append(digits[i:i + 4])

    formatted = " ".join(groups)

    entry.delete(0, tk.END)
    entry.insert(0, formatted)
    entry.icursor(tk.END)


# ==========================================
# BACK BUTTON
# ==========================================

def back_button(command):
    tk.Button(
        window,
        text="برگشت",
        font=("Arial", 14, "bold"),
        bg=LIGHT_GRAY,
        fg=DARK,
        activebackground="#d1d5db",
        relief="flat",
        cursor="hand2",
        command=command
    ).pack(
        padx=55,
        fill="x",
        ipady=9,
        pady=10
    )


# ==========================================
# HOME PAGE
# ==========================================

def home_page():

    clear_page()

    header = tk.Frame(
        window,
        bg=NAVY,
        height=135
    )
    header.pack(fill="x")

    tk.Label(
        header,
        text="بانک ملی",
        font=("Arial", 28, "bold"),
        bg=NAVY,
        fg=WHITE
    ).pack(pady=(25, 5))

    tk.Label(
        header,
        text="ONLINE BANK",
        font=("Arial", 12, "bold"),
        bg=NAVY,
        fg=WHITE
    ).pack()

    balance_box = tk.Frame(
        window,
        bg=WHITE,
        width=370,
        height=145
    )
    balance_box.pack(pady=25)

    balance_box.pack_propagate(False)

    tk.Label(
        balance_box,
        text="موجودی حساب",
        font=("Arial", 17, "bold"),
        bg=WHITE,
        fg=GRAY
    ).pack(pady=(12, 2))

    card_frame = tk.Frame(balance_box, bg=WHITE)
    card_frame.pack(pady=(0, 5))

    tk.Label(
        card_frame,
        text=f"شماره کارت: {ACCOUNT_CARD_NUMBER}",
        font=("Arial", 11, "bold"),
        bg=WHITE,
        fg=DARK
    ).pack(side="left", padx=(0, 6))

    def copy_account_card():
        window.clipboard_clear()
        window.clipboard_append(ACCOUNT_CARD_NUMBER)
        window.update()
        messagebox.showinfo("کپی شد", "شماره کارت با موفقیت کپی شد.")

    tk.Button(
        card_frame,
        text="📋",
        font=("Arial", 12),
        bg=WHITE,
        fg=NAVY,
        activebackground=LIGHT_GRAY,
        relief="flat",
        borderwidth=0,
        cursor="hand2",
        command=copy_account_card
    ).pack(side="left")

    tk.Label(
        balance_box,
        text=f"{money(balance)} تومان",
        font=("Arial", 27, "bold"),
        bg=WHITE,
        fg=NAVY
    ).pack()

    button_style = {
        "font": ("Arial", 16, "bold"),
        "bg": NAVY,
        "fg": WHITE,
        "activebackground": DARK_NAVY,
        "activeforeground": WHITE,
        "relief": "flat",
        "cursor": "hand2"
    }

    tk.Button(
        window,
        text="انتقال پول",
        command=amount_page,
        **button_style
    ).pack(
        padx=55,
        fill="x",
        ipady=10,
        pady=5
    )

    tk.Button(
        window,
        text="شارژ حساب خود",
        command=charge_page,
        **button_style
    ).pack(
        padx=55,
        fill="x",
        ipady=10,
        pady=5
    )

    tk.Button(
        window,
        text="وام فرزند آوری",
        command=loan_page,
        **button_style
    ).pack(
        padx=55,
        fill="x",
        ipady=10,
        pady=5
    )

    tk.Button(
        window,
        text="تراکنش ها",
        command=transactions_page,
        **button_style
    ).pack(
        padx=55,
        fill="x",
        ipady=10,
        pady=5
    )

    tk.Button(
        window,
        text="خروج",
        font=("Arial", 14, "bold"),
        bg=RED,
        fg=WHITE,
        activebackground="#991b1b",
        relief="flat",
        cursor="hand2",
        command=window.destroy
    ).pack(
        padx=55,
        fill="x",
        ipady=9,
        pady=5
    )

    tk.Label(
        window,
        text="ONLINE DEMO",
        font=("Arial", 9),
        bg=BG,
        fg=GRAY
    ).pack(
        side="bottom",
        pady=8
    )


# ==========================================
# CHARGE PAGE
# ==========================================

def charge_page():

    clear_page()

    tk.Label(
        window,
        text="شارژ حساب خود",
        font=("Arial", 26, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(25, 5))

    tk.Label(
        window,
        text="مبلغ شارژ",
        font=("Arial", 17, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(20, 8))

    global charge_amount_entry
    global charge_card_entry
    global charge_cvv_entry
    global charge_expiry_entry

    # Amount
    charge_amount_entry = tk.Entry(
        window,
        font=("Arial", 17),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    charge_amount_entry.pack(
        padx=45,
        fill="x",
        ipady=10
    )

    charge_amount_entry.bind(
        "<KeyRelease>",
        format_money_entry
    )

    # Card number
    tk.Label(
        window,
        text="شماره کارت",
        font=("Arial", 14, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(15, 5))

    charge_card_entry = tk.Entry(
        window,
        font=("Arial", 16),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    charge_card_entry.pack(
        padx=45,
        fill="x",
        ipady=9
    )

    charge_card_entry.bind(
        "<KeyRelease>",
        format_card_entry
    )

    # CVV2
    tk.Label(
        window,
        text="CVV2",
        font=("Arial", 14, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(12, 5))

    charge_cvv_entry = tk.Entry(
        window,
        font=("Arial", 16),
        justify="center",
        show="*",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    charge_cvv_entry.pack(
        padx=45,
        fill="x",
        ipady=9
    )

    # Expiry
    tk.Label(
        window,
        text="تاریخ انقضا",
        font=("Arial", 14, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(12, 5))

    charge_expiry_entry = tk.Entry(
        window,
        font=("Arial", 16),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    charge_expiry_entry.pack(
        padx=45,
        fill="x",
        ipady=9
    )

    # Charge button
    tk.Button(
        window,
        text="شارژ حساب",
        font=("Arial", 16, "bold"),
        bg=NAVY,
        fg=WHITE,
        activebackground=DARK_NAVY,
        relief="flat",
        cursor="hand2",
        command=charge_account
    ).pack(
        padx=55,
        fill="x",
        ipady=10,
        pady=15
    )

    back_button(home_page)


# ==========================================
# CHARGE ACCOUNT
# ==========================================

def charge_account():

    global balance

    amount_text = charge_amount_entry.get()
    amount_text = amount_text.replace(",", "").strip()

    card = charge_card_entry.get()
    card = card.replace(" ", "").strip()

    if card != ACCOUNT_CARD_NUMBER:
        messagebox.showerror(
            "خطا",
            f"برای شارژ حساب فقط باید از شماره کارت {ACCOUNT_CARD_NUMBER} استفاده کنید."
        )
        return

    cvv = charge_cvv_entry.get().strip()
    expiry = charge_expiry_entry.get().strip()

    # Amount check
    if not amount_text.isdigit():
        messagebox.showerror(
            "خطا",
            "مبلغ را به صورت عدد وارد کنید."
        )
        return

    value = int(amount_text)

    if value <= 0:
        messagebox.showerror(
            "خطا",
            "مبلغ باید بیشتر از صفر باشد."
        )
        return

    # Card check
    if not card.isdigit() or len(card) != 16:
        messagebox.showerror(
            "خطا",
            "شماره کارت باید دقیقاً ۱۶ رقم باشد."
        )
        return

    # CVV check
    if not cvv.isdigit() or len(cvv) != 4:
        messagebox.showerror(
            "خطا",
            "CVV2 باید 4 رقم باشد."
        )
        return

    # Expiry check
    if not expiry:
        messagebox.showerror(
            "خطا",
            "تاریخ انقضا را وارد کنید."
        )
        return

    balance += value

    add_transaction(
        "شارژ حساب",
        value,
        card
    )

    save_data()

    messagebox.showinfo(
        "شارژ موفق",
        f"{money(value)} تومان با موفقیت به حساب اضافه شد."
    )

    home_page()


# ==========================================
# AMOUNT PAGE
# ==========================================

def amount_page():

    clear_page()

    tk.Label(
        window,
        text="انتقال پول",
        font=("Arial", 28, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(45, 5))

    tk.Label(
        window,
        text="مرحله ۱ از ۲",
        font=("Arial", 12),
        bg=BG,
        fg=GRAY
    ).pack()

    tk.Label(
        window,
        text="مبلغ انتقال",
        font=("Arial", 19, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(45, 15))

    global amount_entry

    amount_entry = tk.Entry(
        window,
        font=("Arial", 18),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    amount_entry.pack(
        padx=55,
        fill="x",
        ipady=12
    )

    amount_entry.bind(
        "<KeyRelease>",
        format_money_entry
    )

    tk.Label(
        window,
        text=f"موجودی: {money(balance)} تومان",
        font=("Arial", 13),
        bg=BG,
        fg=GRAY
    ).pack(pady=15)

    tk.Button(
        window,
        text="ادامه",
        font=("Arial", 16, "bold"),
        bg=NAVY,
        fg=WHITE,
        activebackground=DARK_NAVY,
        relief="flat",
        cursor="hand2",
        command=check_amount
    ).pack(
        padx=55,
        fill="x",
        ipady=12,
        pady=15
    )

    back_button(home_page)


# ==========================================
# CHECK AMOUNT
# ==========================================

def check_amount():

    global amount

    value = amount_entry.get()
    value = value.replace(",", "").strip()

    if not value.isdigit():
        messagebox.showerror(
            "خطا",
            "لطفاً فقط عدد وارد کنید."
        )
        return

    value = int(value)

    if value <= 0:
        messagebox.showerror(
            "خطا",
            "مبلغ باید بیشتر از صفر باشد."
        )
        return

    if value > balance:
        messagebox.showerror(
            "خطا",
            "موجودی کافی نیست."
        )
        return

    amount = value

    card_page()


# ==========================================
# CARD PAGE
# ==========================================

def card_page():

    clear_page()

    tk.Label(
        window,
        text="شماره کارت مقصد",
        font=("Arial", 25, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(45, 5))

    tk.Label(
        window,
        text="مرحله ۲ از ۲",
        font=("Arial", 12),
        bg=BG,
        fg=GRAY
    ).pack()

    tk.Label(
        window,
        text="شماره کارت ۱۶ رقمی",
        font=("Arial", 18, "bold"),
        bg=BG,
        fg=DARK
    ).pack(pady=(45, 12))

    global card_entry

    card_entry = tk.Entry(
        window,
        font=("Arial", 17),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    card_entry.pack(
        padx=40,
        fill="x",
        ipady=11
    )

    card_entry.bind(
        "<KeyRelease>",
        format_card_entry
    )

    tk.Label(
        window,
        text=f"مبلغ انتقال: {money(amount)} تومان",
        font=("Arial", 13),
        bg=BG,
        fg=GRAY
    ).pack(pady=25)

    tk.Button(
        window,
        text="تأیید انتقال",
        font=("Arial", 16, "bold"),
        bg=NAVY,
        fg=WHITE,
        activebackground=DARK_NAVY,
        relief="flat",
        cursor="hand2",
        command=check_card
    ).pack(
        padx=55,
        fill="x",
        ipady=11,
        pady=8
    )

    back_button(amount_page)


# ==========================================
# CHECK CARD
# ==========================================

def check_card():

    global card_number

    card = card_entry.get()
    card = card.replace(" ", "").strip()

    if not card.isdigit():
        messagebox.showerror(
            "خطا",
            "شماره کارت فقط باید شامل عدد باشد."
        )
        return

    if len(card) != 16:
        messagebox.showerror(
            "خطا",
            "شماره کارت باید دقیقاً ۱۶ رقم باشد."
        )
        return

    card_number = card

    confirm = messagebox.askyesno(
        "تأیید انتقال",
        (
            f"مبلغ:\n{money(amount)} تومان\n\n"
            f"کارت مقصد:\n"
            f"**** **** **** {card[-4:]}\n\n"
            "انتقال انجام  شود؟"
        )
    )

    if confirm:
        transfer()


# ==========================================
# TRANSFER
# ==========================================

def transfer():

    global balance

    balance -= amount

    add_transaction(
        "انتقال پول",
        amount,
        card_number
    )

    save_data()

    success_page()


# ==========================================
# SUCCESS PAGE
# ==========================================

def success_page():

    clear_page()

    now = datetime.now()

    tk.Label(
        window,
        text="✓",
        font=("Arial", 60, "bold"),
        bg=BG,
        fg=GREEN
    ).pack(pady=(30, 0))

    tk.Label(
        window,
        text="انتقال موفق",
        font=("Arial", 27, "bold"),
        bg=BG,
        fg=GREEN
    ).pack()

    tk.Label(
        window,
        text="تراکنش با موفقیت ثبت شد",
        font=("Arial", 13),
        bg=BG,
        fg=GRAY
    ).pack(pady=5)

    receipt = tk.Frame(
        window,
        bg=WHITE,
        width=360,
        height=290
    )
    receipt.pack(pady=20)

    receipt.pack_propagate(False)

    tk.Label(
        receipt,
        text=f"{money(amount)} تومان",
        font=("Arial", 22, "bold"),
        bg=WHITE,
        fg=NAVY
    ).pack(pady=(20, 8))

    tk.Label(
        receipt,
        text=f"کارت مقصد\n**** **** **** {card_number[-4:]}",
        font=("Arial", 13),
        bg=WHITE,
        fg=GRAY
    ).pack(pady=5)

    tk.Label(
        receipt,
        text=f"تاریخ: {now.strftime('%Y/%m/%d')}",
        font=("Arial", 12),
        bg=WHITE,
        fg=GRAY
    ).pack(pady=3)

    tk.Label(
        receipt,
        text=f"زمان: {now.strftime('%H:%M:%S')}",
        font=("Arial", 12),
        bg=WHITE,
        fg=GRAY
    ).pack(pady=3)

    tk.Label(
        receipt,
        text="وضعیت: موفق",
        font=("Arial", 13, "bold"),
        bg=WHITE,
        fg=GREEN
    ).pack(pady=5)

    tk.Label(
        receipt,
        text=f"موجودی جدید: {money(balance)} تومان",
        font=("Arial", 12),
        bg=WHITE,
        fg=DARK
    ).pack(pady=3)

    tk.Button(
        window,
        text="صفحه اصلی",
        font=("Arial", 16, "bold"),
        bg=NAVY,
        fg=WHITE,
        activebackground=DARK_NAVY,
        relief="flat",
        cursor="hand2",
        command=home_page
    ).pack(
        padx=55,
        fill="x",
        ipady=11
    )


# ==========================================
# LOAN PAGE
# ==========================================

def loan_page():
    clear_page()

    tk.Label(
        window,
        text="وام فرزند آوری",
        font=("Arial", 28, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(25, 5))

    tk.Label(
        window,
        text="یکی از گزینه‌های وام را انتخاب کنید",
        font=("Arial", 14),
        bg=BG,
        fg=GRAY
    ).pack(pady=(0, 20))

    if active_loan:
        remaining = active_loan.get("remaining", 0)
        total = active_loan.get("amount", 0)
        interval = active_loan.get("interval_minutes", 0)

        box = tk.Frame(window, bg=WHITE, width=360, height=145)
        box.pack(padx=35, pady=5)
        box.pack_propagate(False)

        tk.Label(
            box,
            text="وام فعال",
            font=("Arial", 17, "bold"),
            bg=WHITE,
            fg=NAVY
        ).pack(pady=(15, 5))

        tk.Label(
            box,
            text=f"اصل وام: {money(total)} تومان",
            font=("Arial", 12),
            bg=WHITE,
            fg=DARK
        ).pack()

        tk.Label(
            box,
            text=f"باقی‌مانده: {money(remaining)} تومان",
            font=("Arial", 12, "bold"),
            bg=WHITE,
            fg=RED
        ).pack()

        tk.Label(
            box,
            text=f"کسر هر {interval} دقیقه: "
                 f"{money(active_loan.get('installment', 0))} تومان",
            font=("Arial", 11),
            bg=WHITE,
            fg=GRAY
        ).pack(pady=3)

        tk.Button(
            window,
            text="برگشت",
            font=("Arial", 14, "bold"),
            bg=LIGHT_GRAY,
            fg=DARK,
            activebackground="#d1d5db",
            relief="flat",
            cursor="hand2",
            command=home_page
        ).pack(padx=55, fill="x", ipady=9, pady=15)

        return

    button_style = {
        "font": ("Arial", 16, "bold"),
        "bg": NAVY,
        "fg": WHITE,
        "activebackground": DARK_NAVY,
        "activeforeground": WHITE,
        "relief": "flat",
        "cursor": "hand2"
    }

    for index, loan_amount in enumerate(LOAN_OPTIONS, start=1):
        tk.Button(
            window,
            text=f"وام {index}: {money(loan_amount)} تومان",
            command=lambda value=loan_amount: select_loan(value),
            **button_style
        ).pack(
            padx=55,
            fill="x",
            ipady=9,
            pady=4
        )

    back_button(home_page)


def select_loan(loan_amount):
    global active_loan

    if active_loan:
        messagebox.showwarning(
            "وام فعال",
            "در حال حاضر یک وام فعال دارید. ابتدا وام فعلی را تسویه کنید."
        )
        return

    interval = ask_loan_interval()
    if interval is None:
        return

    installment = loan_amount // interval
    if installment <= 0:
        messagebox.showerror("خطا", "مبلغ وام برای این زمان کافی نیست.")
        return

    confirm = messagebox.askyesno(
        "تأیید وام",
        f"مبلغ وام: {money(loan_amount)} تومان\n"
        f"کسر از حساب: هر {interval} دقیقه\n"
        f"مبلغ هر کسر: {money(installment)} تومان\n\n"
        "وام دریافت شود؟"
    )

    if not confirm:
        return

    now = datetime.now()
    active_loan = {
        "amount": loan_amount,
        "remaining": loan_amount,
        "interval_minutes": interval,
        "installment": installment,
        "next_payment_at": (
            now.timestamp() + interval * 60
        )
    }

    # مبلغ وام فوراً به موجودی اضافه می‌شود.
    global balance
    balance += loan_amount

    add_transaction("دریافت وام", loan_amount)
    save_data()

    messagebox.showinfo(
        "وام موفق",
        f"{money(loan_amount)} تومان به حساب شما اضافه شد.\n\n"
        f"هر {interval} دقیقه {money(installment)} تومان از حساب کم می‌شود."
    )

    home_page()


def ask_loan_interval():
    result = {"value": None}

    dialog = tk.Toplevel(window)
    dialog.title("زمان بازپرداخت")
    dialog.geometry("360x260")
    dialog.resizable(False, False)
    dialog.configure(bg=BG)
    dialog.transient(window)
    dialog.grab_set()

    tk.Label(
        dialog,
        text="هر چند دقیقه از حسابت کم شود؟",
        font=("Arial", 16, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(25, 10))

    tk.Label(
        dialog,
        text="یک عدد بین ۱ تا ۶۰ دقیقه وارد کنید",
        font=("Arial", 11),
        bg=BG,
        fg=GRAY
    ).pack(pady=5)

    entry = tk.Entry(
        dialog,
        font=("Arial", 18),
        justify="center",
        bg=WHITE,
        fg=DARK,
        relief="solid",
        bd=1
    )
    entry.pack(padx=55, fill="x", ipady=9, pady=12)
    entry.focus_set()

    def confirm_interval():
        value = entry.get().strip()

        if not value.isdigit():
            messagebox.showerror(
                "خطا",
                "لطفاً فقط یک عدد وارد کنید.",
                parent=dialog
            )
            return

        value = int(value)

        if value < 1 or value > 60:
            messagebox.showerror(
                "خطا",
                "زمان باید بین ۱ تا ۶۰ دقیقه باشد.",
                parent=dialog
            )
            return

        result["value"] = value
        dialog.destroy()

    tk.Button(
        dialog,
        text="تأیید",
        font=("Arial", 14, "bold"),
        bg=NAVY,
        fg=WHITE,
        activebackground=DARK_NAVY,
        relief="flat",
        cursor="hand2",
        command=confirm_interval
    ).pack(padx=55, fill="x", ipady=9, pady=5)

    dialog.protocol(
        "WM_DELETE_WINDOW",
        dialog.destroy
    )

    window.wait_window(dialog)
    return result["value"]


def process_loan_payment():
    global active_loan
    global balance

    if not active_loan:
        return

    now_timestamp = datetime.now().timestamp()
    next_payment = active_loan.get("next_payment_at", now_timestamp)

    if now_timestamp < next_payment:
        return

    installment = active_loan.get("installment", 0)
    remaining = active_loan.get("remaining", 0)

    # اگر موجودی کافی باشد، قسط از حساب کم می‌شود.
    if balance >= installment:
        balance -= installment
        remaining -= installment

        add_transaction("پرداخت قسط وام", installment)

        if remaining <= 0:
            messagebox.showinfo(
                "تسویه وام",
                "وام شما به طور کامل تسویه شد."
            )
            active_loan = None
        else:
            active_loan["remaining"] = remaining
            active_loan["next_payment_at"] = (
                now_timestamp
                + active_loan["interval_minutes"] * 60
            )

        save_data()

    else:
        # در صورت کمبود موجودی، قسط حذف نمی‌شود و در اجرای بعدی
        # دوباره تلاش می‌شود.
        active_loan["next_payment_at"] = (
            now_timestamp
            + active_loan["interval_minutes"] * 60
        )
        save_data()

        messagebox.showwarning(
            "موجودی کافی نیست",
            "برای پرداخت قسط وام موجودی کافی نیست.\n"
            "در نوبت بعدی دوباره تلاش می‌شود."
        )


def loan_timer():
    process_loan_payment()
    window.after(1000, loan_timer)


# ==========================================
# DELETE TRANSACTION
# ==========================================

def delete_transaction(index):

    if index < 0 or index >= len(transactions):
        return

    confirm = messagebox.askyesno(
        "حذف تراکنش",
        "آیا می‌خواهید این تراکنش حذف شود؟"
    )

    if not confirm:
        return

    del transactions[index]

    save_data()

    transactions_page()


# ==========================================
# TRANSACTIONS PAGE
# ==========================================

def transactions_page():

    clear_page()

    tk.Label(
        window,
        text="تراکنش ها",
        font=("Arial", 28, "bold"),
        bg=BG,
        fg=NAVY
    ).pack(pady=(20, 10))

    # Scrollable area
    container = tk.Frame(
        window,
        bg=BG
    )
    container.pack(
        fill="both",
        expand=True,
        padx=20
    )

    canvas = tk.Canvas(
        container,
        bg=BG,
        highlightthickness=0
    )
    canvas.pack(
        side="left",
        fill="both",
        expand=True
    )

    scrollbar = tk.Scrollbar(
        container,
        orient="vertical",
        command=canvas.yview
    )
    scrollbar.pack(
        side="right",
        fill="y"
    )

    canvas.configure(
        yscrollcommand=scrollbar.set
    )

    list_frame = tk.Frame(
        canvas,
        bg=BG
    )

    canvas_window = canvas.create_window(
        (0, 0),
        window=list_frame,
        anchor="nw"
    )

    def update_scroll(event=None):
        canvas.configure(
            scrollregion=canvas.bbox("all")
        )

    list_frame.bind(
        "<Configure>",
        update_scroll
    )

    def resize_frame(event):
        canvas.itemconfig(
            canvas_window,
            width=event.width
        )

    canvas.bind(
        "<Configure>",
        resize_frame
    )

    # ======================================
    # NO TRANSACTIONS
    # ======================================

    if not transactions:

        tk.Label(
            list_frame,
            text="هنوز تراکنشی ثبت نشده است.",
            font=("Arial", 15),
            bg=BG,
            fg=GRAY
        ).pack(pady=70)

    else:

        # Show newest first
        for display_index, transaction in enumerate(
            reversed(transactions)
        ):

            real_index = len(transactions) - 1 - display_index

            box = tk.Frame(
                list_frame,
                bg=WHITE,
                height=120
            )

            box.pack(
                fill="x",
                pady=5
            )

            box.pack_propagate(False)

            transaction_type = transaction.get(
                "type",
                "تراکنش"
            )

            transaction_amount = transaction.get(
                "amount",
                0
            )

            date_text = transaction.get(
                "date",
                ""
            )

            time_text = transaction.get(
                "time",
                ""
            )

            status = transaction.get(
                "status",
                "موفق"
            )

            # Type
            tk.Label(
                box,
                text=transaction_type,
                font=("Arial", 14, "bold"),
                bg=WHITE,
                fg=DARK
            ).place(
                x=15,
                y=10
            )

            # Amount
            if transaction_type == "شارژ حساب":
                amount_text = f"+{money(transaction_amount)} تومان"
            else:
                amount_text = f"-{money(transaction_amount)} تومان"

            tk.Label(
                box,
                text=amount_text,
                font=("Arial", 14, "bold"),
                bg=WHITE,
                fg=DARK
            ).place(
                x=15,
                y=38
            )

            # Date
            tk.Label(
                box,
                text=f"{date_text}  |  {time_text}",
                font=("Arial", 9),
                bg=WHITE,
                fg=GRAY
            ).place(
                x=15,
                y=68
            )

            # Status
            tk.Label(
                box,
                text=f"● {status}",
                font=("Arial", 10, "bold"),
                bg=WHITE,
                fg=GREEN
            ).place(
                x=15,
                y=88
            )

            # Card
            card = transaction.get(
                "card",
                ""
            )

            if card:

                tk.Label(
                    box,
                    text=f"کارت: **** **** **** {card[-4:]}",
                    font=("Arial", 9),
                    bg=WHITE,
                    fg=GRAY
                ).place(
                    x=190,
                    y=70
                )

            # Delete button
            tk.Button(
                box,
                text="×",
                font=("Arial", 18, "bold"),
                bg=WHITE,
                fg=RED,
                activebackground=WHITE,
                activeforeground=RED,
                relief="flat",
                borderwidth=0,
                cursor="hand2",
                command=lambda i=real_index:
                    delete_transaction(i)
            ).place(
                relx=1.0,
                x=-12,
                y=8,
                anchor="ne"
            )

    # ======================================
    # BACK BUTTON
    # ======================================

    tk.Button(
        window,
        text="برگشت",
        font=("Arial", 14, "bold"),
        bg=LIGHT_GRAY,
        fg=DARK,
        activebackground="#d1d5db",
        relief="flat",
        cursor="hand2",
        command=home_page
    ).pack(
        padx=55,
        fill="x",
        ipady=9,
        pady=10
    )


# ==========================================
# START
# ==========================================

load_data()
home_page()
window.after(1000, loan_timer)
window.mainloop()
