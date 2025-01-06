stock_trading_project/
├── trading.py           # Kernlogik
├── data/market_data.csv # Simulierte Marktdaten
├── README.md            # Projektübersicht

date,stock,price
2025-01-01,AAPL,150.00
2025-01-02,AAPL,152.00
2025-01-03,AAPL,149.50
2025-01-01,GOOGL,2800.00
2025-01-02,GOOGL,2825.00
2025-01-03,GOOGL,2785.00


import csv
from datetime import datetime

class StockTradingSystem:
    def __init__(self, market_data_path):
        self.market_data = self.load_market_data(market_data_path)
        self.portfolio = {}  # Portfolio: {stock: {"quantity": int, "avg_price": float}}
        self.cash_balance = 10000.00  # Startkapital

    def load_market_data(self, path):
        market_data = []
        with open(path, "r") as file:
            reader = csv.DictReader(file)
            for row in reader:
                market_data.append({
                    "date": datetime.strptime(row["date"], "%Y-%m-%d"),
                    "stock": row["stock"],
                    "price": float(row["price"])
                })
        return market_data

    def get_stock_price(self, stock, date):
        for data in self.market_data:
            if data["stock"] == stock and data["date"] == date:
                return data["price"]
        return None

    def buy_stock(self, stock, quantity, date):
        price = self.get_stock_price(stock, date)
        if price is None:
            return f"Fehler: Keine Daten für {stock} am {date.strftime('%Y-%m-%d')}"

        cost = price * quantity
        if self.cash_balance >= cost:
            self.cash_balance -= cost
            if stock in self.portfolio:
                total_quantity = self.portfolio[stock]["quantity"] + quantity
                total_cost = (self.portfolio[stock]["avg_price"] * self.portfolio[stock]["quantity"]) + cost
                avg_price = total_cost / total_quantity
                self.portfolio[stock] = {"quantity": total_quantity, "avg_price": avg_price}
            else:
                self.portfolio[stock] = {"quantity": quantity, "avg_price": price}
            return f"Gekauft: {quantity} {stock} zu {price:.2f} (Gesamtkosten: {cost:.2f})"
        else:
            return "Fehler: Nicht genug Guthaben"

    def sell_stock(self, stock, quantity, date):
        if stock not in self.portfolio or self.portfolio[stock]["quantity"] < quantity:
            return f"Fehler: Nicht genug {stock} im Portfolio"

        price = self.get_stock_price(stock, date)
        if price is None:
            return f"Fehler: Keine Daten für {stock} am {date.strftime('%Y-%m-%d')}"

        revenue = price * quantity
        self.cash_balance += revenue
        self.portfolio[stock]["quantity"] -= quantity

        if self.portfolio[stock]["quantity"] == 0:
            del self.portfolio[stock]

        return f"Verkauft: {quantity} {stock} zu {price:.2f} (Gesamterlös: {revenue:.2f})"

    def show_portfolio(self):
        print("Portfolio:")
        for stock, details in self.portfolio.items():
            print(f"{stock}: {details['quantity']} Aktien, Durchschnittspreis: {details['avg_price']:.2f}")
        print(f"Guthaben: {self.cash_balance:.2f}")

# Hauptprogramm
if __name__ == "__main__":
    system = StockTradingSystem("data/market_data.csv")

    # Beispielaktionen
    today = datetime(2025, 1, 2)
    print(system.buy_stock("AAPL", 10, today))
    print(system.buy_stock("GOOGL", 2, today))
    system.show_portfolio()

    # Verkauf
    tomorrow = datetime(2025, 1, 3)
    print(system.sell_stock("AAPL", 5, tomorrow))
    system.show_portfolio()
