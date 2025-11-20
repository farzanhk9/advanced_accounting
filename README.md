import csv
import os
from collections import defaultdict
from datetime import datetime

# File Paths
TRANSACTION_FILE = "transactions.csv"
INVENTORY_FILE = "inventory.csv"

# Categories
INCOME_CATEGORIES = ["sales", "other_income"]
EXPENSE_CATEGORIES = ["inventory_purchase", "rent", "salaries", "other_expenses"]

# Load transactions from CSV
def load_transactions():
    transactions = []
    if not os.path.exists(TRANSACTION_FILE):
        return transactions
    with open(TRANSACTION_FILE, mode="r", newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            transactions.append({
                'date': row['date'],
                'type': row['type'],
                'category': row['category'],
                'amount': float(row['amount']),
                'description': row['description']
            })
    return transactions

# Save transactions to CSV
def save_transactions(transactions):
    with open(TRANSACTION_FILE, mode="w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["date", "type", "category", "amount", "description"])
        writer.writeheader()
        for transaction in transactions:
            writer.writerow(transaction)

# Load inventory from CSV
def load_inventory():
    inventory = defaultdict(int)
    if not os.path.exists(INVENTORY_FILE):
        return inventory
    with open(INVENTORY_FILE, mode="r", newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            inventory[row['item']] = int(row['quantity'])
    return inventory

# Save inventory to CSV
def save_inventory(inventory):
    with open(INVENTORY_FILE, mode="w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["item", "quantity"])
        writer.writeheader()
        for item, quantity in inventory.items():
            writer.writerow({"item": item, "quantity": quantity})

# Add a new transaction
def add_transaction(transactions, inventory):
    date_str = input("Date (YYYY-MM-DD) [leave empty for today]: ").strip()
    if not date_str:
        date_str = datetime.today().strftime("%Y-%m-%d")
    t_type = input("Type (income/expense): ").strip().lower()
    if t_type not in ["income", "expense"]:
        print("Invalid type. Choose 'income' or 'expense'.")
        return

    category = input(f"Category ({', '.join(INCOME_CATEGORIES if t_type == 'income' else EXPENSE_CATEGORIES)}): ").strip()
    if category not in (INCOME_CATEGORIES if t_type == 'income' else EXPENSE_CATEGORIES):
        print(f"Invalid category. Choose from: {', '.join(INCOME_CATEGORIES if t_type == 'income' else EXPENSE_CATEGORIES)}")
        return

    description = input("Description (optional): ").strip()
    try:
        amount = float(input("Amount: ").strip())
    except ValueError:
        print("Invalid amount.")
        return

    if amount <= 0:
        print("Amount must be positive.")
        return

    if t_type == "expense" and category == "inventory_purchase":
        item = input("Item being purchased: ").strip()
        quantity = int(input("Quantity of item: ").strip())
        inventory[item] += quantity
        save_inventory(inventory)

    transaction = {
        'date': date_str,
        'type': t_type,
        'category': category,
        'amount': amount,
        'description': description
    }
    transactions.append(transaction)
    save_transactions(transactions)
    print("Transaction saved.")

# View transactions
def view_transactions(transactions):
    print(f"\n{'Date':<12} {'Type':<8} {'Category':<20} {'Amount':<10} {'Description':<20}")
    print("-" * 70)
    for t in transactions:
        print(f"{t['date']:<12} {t['type']:<8} {t['category']:<20} {t['amount']:<10.2f} {t['description']:<20}")
    print("-" * 70)

# Summary of income, expense, and balance
def summary(transactions):
    total_income = sum(t['amount'] for t in transactions if t['type'] == "income")
    total_expense = sum(t['amount'] for t in transactions if t['type'] == "expense")
    balance = total_income - total_expense
    print(f"\nTotal income: {total_income:,.2f}")
    print(f"Total expenses: {total_expense:,.2f}")
    print(f"Net profit: {balance:,.2f}")

# Summary by category
def summary_by_category(transactions):
    category_income = defaultdict(float)
    category_expense = defaultdict(float)

    for t in transactions:
        if t['type'] == 'income':
            category_income[t['category']] += t['amount']
        else:
            category_expense[t['category']] += t['amount']

    print("\nCategory-wise Summary:")
    print(f"{'Category':<20} {'Income':<15} {'Expense':<15} {'Net':<15}")
    print("-" * 65)

    categories = set(list(category_income.keys()) + list(category_expense.keys()))
    for category in categories:
        income = category_income[category]
        expense = category_expense[category]
        net = income - expense
        print(f"{category:<20} {income:,.2f} {expense:,.2f} {net:,.2f}")

def menu():
    transactions = load_transactions()
    inventory = load_inventory()

    while True:
        print("\n===== Simple Accounting System =====")
        print("1) Add transaction")
        print("2) View all transactions")
        print("3) View transaction summary")
        print("4) View category summary")
        print("5) View current inventory")
        print("0) Exit")
        choice = input("Choose an option: ").strip()

        if choice == "1":
            add_transaction(transactions, inventory)
        elif choice == "2":
            view_transactions(transactions)
        elif choice == "3":
            summary(transactions)
        elif choice == "4":
            summary_by_category(transactions)
        elif choice == "5":
            print("\nInventory Summary:")
            for item, quantity in inventory.items():
                print(f"{item}: {quantity} units")
        elif choice == "0":
            print("Goodbye!")
            break
        else:
            print("Invalid option, try again.")

if __name__ == "__main__":
    menu()
