CODE BÀI TẬP LỚN

Blackjack.py
import tkinter as tk
from tkinter import messagebox, simpledialog
from cards import Deck
from games import Game
import os
from PIL import Image, ImageTk

class BlackjackGUI(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Blackjack")
        self.configure(bg="white")  # Đổi màu nền cửa sổ thành trắng
        self.deck = Deck()
        self.game = Game(self.deck)

        # Thống kê
        self.win_count = 0
        self.lose_count = 0
        self.draw_count = 0
        self.balance = 1000
        self.bet = 0
        self.history = []

        # Style options
        self.card_bg = "white"      # Đổi màu nền lá bài thành trắng
        self.card_fg = "#ffd369"
        self.label_fg = "#222831"   # Đổi màu chữ cho dễ nhìn trên nền trắng
        self.btn_bg = "#222831"
        self.btn_fg = "#ffd369"
        self.status_fg = "#00adb5"
        self.font_main = ("Segoe UI", 14, "bold")
        self.font_card = ("Consolas", 16, "bold")

        # Menu hướng dẫn
        menubar = tk.Menu(self)
        helpmenu = tk.Menu(menubar, tearoff=0)
        helpmenu.add_command(label="Luật chơi", command=self.show_rules)
        menubar.add_cascade(label="Trợ giúp", menu=helpmenu)
        self.config(menu=menubar)

        # Frame Dealer
        tk.Label(self, text="Dealer's Hand", font=self.font_main, fg=self.label_fg, bg="white").pack(pady=(10,0))
        self.dealer_frame = tk.Frame(self, bg="white")
        self.dealer_frame.pack(pady=5)

        # Frame Player
        tk.Label(self, text="Your Hand", font=self.font_main, fg=self.label_fg, bg="white").pack(pady=(20,0))
        self.player_frame = tk.Frame(self, bg="white")
        self.player_frame.pack(pady=5)

        # Status
        self.status = tk.Label(self, text="Game started. Hit or Stand?", font=("Segoe UI", 12, "bold"), fg=self.status_fg, bg="white")
        self.status.pack(pady=10)

        # Thông tin cược và thống kê
        self.info_frame = tk.Frame(self, bg="white")
        self.info_frame.pack(pady=5)
        self.balance_label = tk.Label(self.info_frame, text=f"Balance: ${self.balance}", font=("Segoe UI", 12), fg="#00adb5", bg="white")
        self.balance_label.grid(row=0, column=0, padx=10)
        self.bet_label = tk.Label(self.info_frame, text=f"Bet: ${self.bet}", font=("Segoe UI", 12), fg="#ffd369", bg="white")
        self.bet_label.grid(row=0, column=1, padx=10)
        self.score_label = tk.Label(self.info_frame, text=self.get_score_text(), font=("Segoe UI", 12), fg="#222831", bg="white")
        self.score_label.grid(row=0, column=2, padx=10)

        # Buttons
        self.button_frame = tk.Frame(self, bg="white")
        self.button_frame.pack(pady=10)
        self.bet_button = tk.Button(self.button_frame, text="Bet", command=self.place_bet, font=self.font_main, bg="#00adb5", fg="#222831", width=8, relief="raised", bd=3)
        self.bet_button.grid(row=0, column=0, padx=10)
        self.hit_button = tk.Button(self.button_frame, text="Hit", command=self.hit, font=self.font_main, bg=self.btn_bg, fg=self.btn_fg, width=8, relief="raised", bd=3, activebackground="#393e46", state=tk.DISABLED)
        self.hit_button.grid(row=0, column=1, padx=10)
        self.stand_button = tk.Button(self.button_frame, text="Stand", command=self.stand, font=self.font_main, bg=self.btn_bg, fg=self.btn_fg, width=8, relief="raised", bd=3, activebackground="#393e46", state=tk.DISABLED)
        self.stand_button.grid(row=0, column=2, padx=10)
        self.restart_button = tk.Button(self.button_frame, text="Restart", command=self.restart, font=self.font_main, bg="#ffd369", fg="#222831", width=8, relief="raised", bd=3, activebackground="#393e46")
        self.restart_button.grid(row=0, column=3, padx=10)
        self.history_button = tk.Button(self.button_frame, text="History", command=self.show_history, font=self.font_main, bg="#393e46", fg="#ffd369", width=8, relief="raised", bd=3)
        self.history_button.grid(row=0, column=4, padx=10)

        self.update_display()

    def get_score_text(self):
        return f"Wins: {self.win_count}  Losses: {self.lose_count}  Draws: {self.draw_count}"

    def get_card_image(self, card):
        rank_map = {
            'A': 'ace', 'J': 'jack', 'Q': 'queen', 'K': 'king',
            '10': '10', '9': '9', '8': '8', '7': '7', '6': '6',
            '5': '5', '4': '4', '3': '3', '2': '2'
        }
        suit_map = {
            '♠': 'spades', '♥': 'hearts', '♦': 'diamonds', '♣': 'clubs'
        }
        rank = rank_map[card.rank]
        suit = suit_map[card.suit]
        filename = os.path.join("cards", f"{rank}_of_{suit}.png")
        if not os.path.exists(filename):
            filename = os.path.join("cards", "back.png")  # Ảnh mặc định nếu thiếu
        img = Image.open(filename).resize((60, 90))
        return ImageTk.PhotoImage(img)

    def update_display(self):
        # Xóa các widget cũ
        for widget in self.player_frame.winfo_children():
            widget.destroy()
        for widget in self.dealer_frame.winfo_children():
            widget.destroy()

        # Hiển thị bài người chơi
        self.player_card_imgs = []
        for card in self.game.player.cards:
            img = self.get_card_image(card)
            self.player_card_imgs.append(img)
            tk.Label(self.player_frame, image=img, bg=self.card_bg).pack(side=tk.LEFT, padx=2)

        # Hiển thị bài dealer
        self.dealer_card_imgs = []
        if self.bet == 0:
            # Chưa đặt cược, ẩn toàn bộ bài dealer
            for _ in self.game.dealer.cards:
                try:
                    back_img = ImageTk.PhotoImage(Image.open(os.path.join("cards", "back.png")).resize((60, 90)))
                except:
                    back_img = None
                self.dealer_card_imgs.append(back_img)
                tk.Label(self.dealer_frame, image=back_img, bg=self.card_bg).pack(side=tk.LEFT, padx=2)
        elif self.hit_button['state'] == tk.NORMAL:
            # Đang chơi, chỉ lật 1 lá đầu
            if self.game.dealer.cards:
                img = self.get_card_image(self.game.dealer.cards[0])
                self.dealer_card_imgs.append(img)
                tk.Label(self.dealer_frame, image=img, bg=self.card_bg).pack(side=tk.LEFT, padx=2)
                for _ in self.game.dealer.cards[1:]:
                    try:
                        back_img = ImageTk.PhotoImage(Image.open(os.path.join("cards", "back.png")).resize((60, 90)))
                    except:
                        back_img = None
                    self.dealer_card_imgs.append(back_img)
                    tk.Label(self.dealer_frame, image=back_img, bg=self.card_bg).pack(side=tk.LEFT, padx=2)
        else:
            # Kết thúc ván, lật toàn bộ bài dealer
            for card in self.game.dealer.cards:
                img = self.get_card_image(card)
                self.dealer_card_imgs.append(img)
                tk.Label(self.dealer_frame, image=img, bg=self.card_bg).pack(side=tk.LEFT, padx=2)

        self.balance_label.config(text=f"Balance: ${self.balance}")
        self.bet_label.config(text=f"Bet: ${self.bet}")
        self.score_label.config(text=self.get_score_text())

    def hand_str(self, hand):
        return ' '.join(str(card) for card in hand.cards)

    def place_bet(self):
        if self.bet > 0:
            messagebox.showinfo("Bet", "Bạn đã đặt cược rồi!")
            return
        bet = simpledialog.askinteger("Bet", f"Balance: ${self.balance}\nNhập số tiền cược:", minvalue=1, maxvalue=self.balance)
        if bet:
            self.bet = bet
            self.balance -= bet
            self.status.config(text="Bet placed. Hit or Stand?", fg="#ffd369")
            self.hit_button.config(state=tk.NORMAL)
            self.stand_button.config(state=tk.NORMAL)
            self.bet_button.config(state=tk.DISABLED)
            self.update_display()

    def hit(self):
        self.game.hit(self.game.player)
        self.update_display()
        if self.game.player.value() > 21:
            self.end_game("You busted!")
        elif self.game.player.value() == 21:
            self.end_game("Blackjack!")

    def stand(self):
        while self.game.dealer.value() < 17:
            self.game.hit(self.game.dealer)
        self.reveal_dealer()

        p_val = self.game.player.value()
        d_val = self.game.dealer.value()

        if d_val > 21 or p_val > d_val:
            self.end_game("You win!")
        elif p_val == d_val:
            self.end_game("Draw!")
        else:
            self.end_game("Dealer wins.")

    def reveal_dealer(self):
        self.update_display()

    def end_game(self, result):
        self.reveal_dealer()
        self.status.config(text=result)
        self.hit_button.config(state=tk.DISABLED)
        self.stand_button.config(state=tk.DISABLED)
        self.bet_button.config(state=tk.NORMAL)
        # Cập nhật thống kê và tiền
        if result == "You win!":
            self.win_count += 1
            self.balance += self.bet * 2
        elif result == "Draw!":
            self.draw_count += 1
            self.balance += self.bet
        else:
            self.lose_count += 1
        self.history.append(f"Bet: ${self.bet} - {result} (Balance: ${self.balance})")
        self.bet = 0
        self.deck = Deck()
        self.game = Game(self.deck)
        self.update_display()
        messagebox.showinfo("Game Over", result)

    def restart(self):
        self.deck = Deck()
        self.game = Game(self.deck)
        self.status.config(text="Game started. Place your bet!", fg="#00adb5")
        self.hit_button.config(state=tk.DISABLED)
        self.stand_button.config(state=tk.DISABLED)
        self.bet_button.config(state=tk.NORMAL)
        self.bet = 0
        self.update_display()

    def show_history(self):
        if not self.history:
            messagebox.showinfo("History", "Chưa có lịch sử ván chơi.")
        else:
            messagebox.showinfo("History", "\n".join(self.history[-10:]))

    def show_rules(self):
        rules = (
            "Luật chơi Blackjack:\n"
            "- Mục tiêu: Có tổng điểm gần 21 nhất nhưng không vượt quá 21.\n"
            "- J, Q, K tính 10 điểm. A tính 1 hoặc 11 điểm.\n"
            "- Đặt cược, sau đó chọn Hit (rút bài) hoặc Stand (dừng).\n"
            "- Nếu bạn vượt quá 21 điểm, bạn thua.\n"
            "- Dealer rút đến khi >= 17 điểm.\n"
            "- So điểm để xác định thắng/thua/hòa.\n"
            "- Nếu thắng, bạn nhận lại gấp đôi tiền cược."
        )
        messagebox.showinfo("Luật chơi", rules)

if __name__ == "__main__":
    app = BlackjackGUI()
    app.mainloop()

    Cards.py
    import random


class Card:
    RANKS = ['A'] + [str(n) for n in range(2, 11)] + ['J', 'Q', 'K']
    SUITS = ['♠', '♥', '♦', '♣']

    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit

    def __str__(self):
        return f'{self.rank}{self.suit}'

    def value(self):
        if self.rank in ['J', 'Q', 'K']:
            return 10
        if self.rank == 'A':
            return 11
        return int(self.rank)

class Deck:
    def __init__(self):
        self.reset()

    def reset(self):
        self.cards = [Card(rank, suit) for suit in Card.SUITS for rank in Card.RANKS]
        random.shuffle(self.cards)

    def deal(self):
        if not self.cards:
            self.reset()
        return self.cards.pop() if self.cards else None

Games.py
from cards import Deck, Card

class Hand:
    def __init__(self):
        self.cards = []

    def add(self, card):
        self.cards.append(card)

    def value(self):
        val = sum(card.value() for card in self.cards)
        num_aces = sum(1 for card in self.cards if card.rank == 'A')
        while val > 21 and num_aces:
            val -= 10
            num_aces -= 1
        return val

    def __str__(self):
        return ' '.join(str(card) for card in self.cards)

class Game:
    def __init__(self, deck):
        self.deck = deck
        self.player = Hand()
        self.dealer = Hand()
        self.deal_initial()

    def deal_initial(self):
        for _ in range(2):
            card_p = self.deck.deal()
            card_d = self.deck.deal()
            if card_p: self.player.add(card_p)
            if card_d: self.dealer.add(card_d)

    def hit(self, hand):
        card = self.deck.deal()
        if card:
            hand.add(card)

    def reset(self):
        self.deck.reset()
        self.player = Hand()
        self.dealer = Hand()
        self.deal_initial()
        
