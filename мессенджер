import socket
import threading
import tkinter as tk
from tkinter import scrolledtext, messagebox
import datetime
import time

class SimpleMessenger:
    def __init__(self):
        self.host = self.get_local_ip()
        self.port = 8888
        self.clients = {}
        self.running = True
        
        print(f"Ваш IP: {self.host}")
        
        self.setup_gui()
        self.start_network()
    
    def get_local_ip(self):
        """Получаем локальный IP"""
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            s.connect(("8.8.8.8", 80))
            ip = s.getsockname()[0]
            s.close()
            return ip
        except:
            try:
                return socket.gethostbyname(socket.gethostname())
            except:
                return "127.0.0.1"
    
    def start_network(self):
        """Запускаем сетевые компоненты"""
        # UDP сервер для приема сообщений
        self.udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.udp_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.udp_socket.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        
        try:
            self.udp_socket.bind(('', self.port))
            self.add_message(f"✅ Сервер запущен на порту {self.port}", "system")
        except Exception as e:
            self.add_message(f"❌ Ошибка: {e}", "error")
            return
        
        # Запускаем потоки
        threading.Thread(target=self.receive_messages, daemon=True).start()
        threading.Thread(target=self.broadcast_presence, daemon=True).start()
        threading.Thread(target=self.scan_network, daemon=True).start()
    
    def broadcast_presence(self):
        """Регулярная отправка broadcast сообщений"""
        while self.running:
            try:
                message = f"HELLO:{self.host}"
                self.udp_socket.sendto(message.encode('utf-8'), ('<broadcast>', self.port))
                time.sleep(3)  # Отправляем каждые 3 секунды
            except Exception as e:
                time.sleep(5)
    
    def scan_network(self):
        """Активное сканирование сети"""
        base_ip = ".".join(self.host.split('.')[:-1]) + "."
        
        while self.running:
            try:
                # Сканируем диапазон 1-254
                for i in range(1, 255):
                    if not self.running:
                        break
                    target_ip = f"{base_ip}{i}"
                    if target_ip != self.host:
                        try:
                            message = f"PING:{self.host}"
                            self.udp_socket.sendto(message.encode('utf-8'), (target_ip, self.port))
                        except:
                            pass
                time.sleep(10)  # Сканируем каждые 10 секунд
            except:
                time.sleep(10)
    
    def receive_messages(self):
        """Прием сообщений"""
        while self.running:
            try:
                data, addr = self.udp_socket.recvfrom(1024)
                message = data.decode('utf-8')
                self.handle_message(message, addr[0])
            except:
                pass
    
    def handle_message(self, message, ip):
        """Обработка входящих сообщений"""
        if message.startswith("HELLO:"):
            user_ip = message.split(":")[1]
            if user_ip != self.host and user_ip not in self.clients:
                self.clients[user_ip] = ip
                self.update_users_list()
                self.add_message(f"✅ Обнаружен: {user_ip}", "system")
                # Отвечаем на приветствие
                response = f"HELLO:{self.host}"
                self.udp_socket.sendto(response.encode('utf-8'), (ip, self.port))
        
        elif message.startswith("PING:"):
            user_ip = message.split(":")[1]
            if user_ip != self.host:
                # Отвечаем на ping
                response = f"HELLO:{self.host}"
                self.udp_socket.sendto(response.encode('utf-8'), (ip, self.port))
        
        elif message.startswith("MSG:"):
            # Обычное сообщение
            parts = message.split(":", 2)
            if len(parts) == 3:
                sender, content = parts[1], parts[2]
                self.add_message(f"[{sender}] {content}", "normal")
        
        elif message.startswith("PRIVATE:"):
            # Личное сообщение
            parts = message.split(":", 3)
            if len(parts) == 4:
                sender, target, content = parts[1], parts[2], parts[3]
                if target == self.host:
                    self.add_message(f"[Лично от {sender}] {content}", "private")
    
    def send_message(self, event=None):
        """Отправка сообщения"""
        message = self.message_entry.get().strip()
        if not message:
            return
        
        # Отправляем всем пользователям
        for user_ip in self.clients:
            try:
                msg = f"MSG:{self.host}:{message}"
                self.udp_socket.sendto(msg.encode('utf-8'), (user_ip, self.port))
            except:
                pass
        
        self.add_message(f"[Я] {message}", "my_message")
        self.message_entry.delete(0, tk.END)
    
    def send_private_message(self):
        """Отправка личного сообщения"""
        selection = self.users_listbox.curselection()
        if not selection:
            messagebox.showwarning("Внимание", "Выберите пользователя из списка")
            return
        
        target_ip = self.users_listbox.get(selection[0])
        message = self.private_entry.get().strip()
        if not message:
            return
        
        try:
            msg = f"PRIVATE:{self.host}:{target_ip}:{message}"
            self.udp_socket.sendto(msg.encode('utf-8'), (target_ip, self.port))
            self.add_message(f"[Я → {target_ip}] {message}", "my_private")
            self.private_entry.delete(0, tk.END)
        except Exception as e:
            self.add_message(f"❌ Ошибка отправки: {e}", "error")
    
    def manual_connect(self):
        """Ручное подключение по IP"""
        ip = self.manual_ip_entry.get().strip()
        if not ip:
            return
        
        if ip == self.host:
            messagebox.showwarning("Внимание", "Нельзя подключиться к себе")
            return
        
        if ip not in self.clients:
            self.clients[ip] = ip
            self.update_users_list()
            self.add_message(f"✅ Ручное подключение: {ip}", "system")
            
            # Отправляем приветствие
            try:
                message = f"HELLO:{self.host}"
                self.udp_socket.sendto(message.encode('utf-8'), (ip, self.port))
            except:
                pass
        
        self.manual_ip_entry.delete(0, tk.END)
    
    def setup_gui(self):
        """Настройка интерфейса"""
        self.root = tk.Tk()
        self.root.title(f"Простой мессенджер - {self.host}")
        self.root.geometry("900x600")
        
        # Основной фрейм
        main_frame = tk.Frame(self.root)
        main_frame.pack(fill='both', expand=True, padx=10, pady=10)
        
        # Левая часть - чат
        left_frame = tk.Frame(main_frame)
        left_frame.pack(side='left', fill='both', expand=True)
        
        # Область чата
        chat_frame = tk.LabelFrame(left_frame, text="Чат")
        chat_frame.pack(fill='both', expand=True)
        
        self.chat_text = scrolledtext.ScrolledText(
            chat_frame,
            wrap=tk.WORD,
            state='disabled',
            height=20
        )
        self.chat_text.pack(fill='both', expand=True, padx=5, pady=5)
        
        # Ввод сообщения
        input_frame = tk.Frame(left_frame)
        input_frame.pack(fill='x', pady=5)
        
        tk.Label(input_frame, text="Сообщение для всех:").pack(anchor='w')
        self.message_entry = tk.Entry(input_frame, width=50)
        self.message_entry.pack(fill='x', pady=2)
        self.message_entry.bind('<Return>', self.send_message)
        
        send_btn = tk.Button(input_frame, text="Отправить всем", command=self.send_message)
        send_btn.pack(pady=2)
        
        # Правая часть - управление
        right_frame = tk.Frame(main_frame)
        right_frame.pack(side='right', fill='y', padx=(10, 0))
        
        # Ручное подключение
        manual_frame = tk.LabelFrame(right_frame, text="Ручное подключение")
        manual_frame.pack(fill='x', pady=(0, 10))
        
        tk.Label(manual_frame, text="IP адрес:").pack(anchor='w')
        self.manual_ip_entry = tk.Entry(manual_frame)
        self.manual_ip_entry.pack(fill='x', pady=2)
        
        manual_btn = tk.Button(manual_frame, text="Подключиться", command=self.manual_connect)
        manual_btn.pack(fill='x', pady=2)
        
        # Список пользователей
        users_frame = tk.LabelFrame(right_frame, text="Пользователи онлайн")
        users_frame.pack(fill='x', pady=(0, 10))
        
        self.users_listbox = tk.Listbox(users_frame, height=10)
        self.users_listbox.pack(fill='both', expand=True, padx=5, pady=5)
        
        # Личные сообщения
        private_frame = tk.LabelFrame(right_frame, text="Личное сообщение")
        private_frame.pack(fill='x')
        
        tk.Label(private_frame, text="Сообщение:").pack(anchor='w')
        self.private_entry = tk.Entry(private_frame)
        self.private_entry.pack(fill='x', pady=2)
        
        private_btn = tk.Button(private_frame, text="Отправить выбранному", command=self.send_private_message)
        private_btn.pack(fill='x', pady=2)
        
        # Кнопки управления
        control_frame = tk.Frame(right_frame)
        control_frame.pack(fill='x', pady=10)
        
        refresh_btn = tk.Button(control_frame, text="Обновить список", command=self.force_refresh)
        refresh_btn.pack(fill='x', pady=2)
        
        clear_btn = tk.Button(control_frame, text="Очистить чат", command=self.clear_chat)
        clear_btn.pack(fill='x', pady=2)
        
        # Запускаем обновление интерфейса
        self.root.after(1000, self.update_interface)
    
    def force_refresh(self):
        """Принудительное обновление списка пользователей"""
        self.add_message("🔄 Принудительный поиск...", "system")
        # Отправляем broadcast
        try:
            message = f"HELLO:{self.host}"
            self.udp_socket.sendto(message.encode('utf-8'), ('<broadcast>', self.port))
        except:
            pass
    
    def clear_chat(self):
        """Очистка чата"""
        self.chat_text.config(state='normal')
        self.chat_text.delete(1.0, tk.END)
        self.chat_text.config(state='disabled')
    
    def add_message(self, message, msg_type="normal"):
        """Добавление сообщения в чат"""
        self.chat_text.config(state='normal')
        
        colors = {
            "system": "blue",
            "error": "red",
            "private": "purple", 
            "my_private": "dark violet",
            "my_message": "green",
            "normal": "black"
        }
        
        timestamp = datetime.datetime.now().strftime("%H:%M:%S")
        formatted_message = f"[{timestamp}] {message}\n"
        
        # Добавляем сообщение
        self.chat_text.insert(tk.END, formatted_message)
        
        # Применяем цвет
        if msg_type in colors:
            start_index = f"{self.chat_text.index(tk.END)} - {len(formatted_message) + 1}c"
            end_index = self.chat_text.index(tk.END)
            self.chat_text.tag_add(msg_type, start_index, end_index)
            self.chat_text.tag_config(msg_type, foreground=colors[msg_type])
        
        self.chat_text.config(state='disabled')
        self.chat_text.see(tk.END)
    
    def update_users_list(self):
        """Обновление списка пользователей"""
        self.users_listbox.delete(0, tk.END)
        for user_ip in sorted(self.clients.keys()):
            self.users_listbox.insert(tk.END, user_ip)
    
    def update_interface(self):
        """Периодическое обновление интерфейса"""
        self.update_users_list()
        self.root.after(2000, self.update_interface)
    
    def on_closing(self):
        """Действия при закрытии"""
        self.running = False
        try:
            self.udp_socket.close()
        except:
            pass
        self.root.destroy()
    
    def run(self):
        """Запуск приложения"""
        self.root.protocol("WM_DELETE_WINDOW", self.on_closing)
        self.root.mainloop()

if __name__ == "__main__":
    app = SimpleMessenger()
    app.run()
