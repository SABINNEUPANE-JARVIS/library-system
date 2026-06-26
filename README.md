import json
import string
import random
from pathlib import Path
 
 
class Librarysystem:
    file = 'data.json'
    storage = []
    books = {
        "basic": [
            {"id": "B001", "title": "Python Basics", "available": True},
            {"id": "B002", "title": "JavaScript 101", "available": True},
            {"id": "B003", "title": "HTML & CSS", "available": False},
        ],
        "premium": [
            {"id": "P001", "title": "Advanced ML", "available": True},
            {"id": "P002", "title": "Deep Learning", "available": True},
            {"id": "P003", "title": "AI Ethics", "available": False},
            {"id": "P004", "title": "Computer Vision", "available": True},
        ]
    }
 
    @classmethod
    def load_data(cls):
        if Path(cls.file).exists():
            with open(cls.file) as fs:
                cls.storage = json.loads(fs.read())
 
    @classmethod
    def update_data(cls):
        with open(cls.file, 'w') as fs:
            fs.write(json.dumps(cls.storage))
 
    @classmethod
    def __accountgenerator(cls):
        alpha = random.choices(string.ascii_letters, k=3)
        num = random.choices(string.digits, k=2)
        Id = alpha + num
        random.shuffle(Id)
        return ''.join(Id)
 
    def createaccount(self):
        # FIX: ask email once and reuse it
        email = input("Tell your email: ").strip()
 
        account_exists = any(acc['email'] == email for acc in Librarysystem.storage)
 
        if account_exists:
            print("❌ This account already exists. Please login.")
            return None
 
        print("Choose membership type:")
        account_type = input("Basic or Premium: ").strip().lower()
 
        if account_type not in ['basic', 'premium']:
            print("❌ Account type must be 'basic' or 'premium'")
            return None
 
        # FIX: set fee based on account type
        fee = 30 if account_type == 'basic' else 50
        print(f"Please pay ${fee} for {account_type} membership.")
 
        try:
            amount = int(input(f"Enter amount: $"))
        except ValueError:
            print("❌ Please enter a valid amount.")
            return None
 
        if amount != fee:
            print(f"❌ Incorrect amount. Please pay ${fee}.")
            return None
 
        # FIX: collect details once, reuse email from above
        try:
            details = {
                "name": input("Tell your name: ").strip(),
                "age": int(input("Tell your age: ")),
                "email": email,  # FIX: reuse email, don't ask twice
                "pin": int(input("Tell your 4-digit PIN: ")),
                "account_type": account_type,  # FIX: save account type
                "accountNo.": Librarysystem.__accountgenerator(),
                "borrowed_books": []  # FIX: track borrowed books
            }
        except ValueError:
            print("❌ Please enter valid details.")
            return None
 
        if details['age'] < 10:
            print("❌ You cannot create an account. Please ask a parent to create one.")
            return None
 
        if len(str(details['pin'])) != 4:
            print("❌ PIN must be exactly 4 digits.")
            return None
 
        Librarysystem.storage.append(details)
        Librarysystem.update_data()
        print(f"✅ Account created successfully!")
        print(f"   Your Account Number: {details['accountNo.']}")
 
    def login(self):
        accnumber = input("Please tell your account number: ").strip()
        try:
            pin = int(input("Please tell your PIN: "))
        except ValueError:
            print("❌ PIN must be a number.")
            return None
 
        userdata = [i for i in Librarysystem.storage
                    if i["accountNo."] == accnumber and i["pin"] == pin]
 
        if not userdata:
            print("❌ Invalid account number or PIN.")
            return None
 
        self.current_user = userdata[0]
        print(f"✅ Login successful! Welcome {self.current_user['name']}")
        self.showdetails()
        self.post_login_menu()
        return userdata[0]
 
    def post_login_menu(self):
        while True:
            print("\n" + "=" * 50)
            print(f"📚 WELCOME {self.current_user['name'].upper()} 📚")
            print("=" * 40)
            print("1. Borrow Books")
            print("2. Return Books")
            print("3. Logout")
            print("=" * 40)
            choice = input("Enter choice: ").strip()
 
            if choice == '1':
                self.borrowbooks()
            elif choice == '2':
                self.returnbooks()
            elif choice == '3':
                print("👋 Logging out...")
                break
            else:
                print("❌ Invalid choice")
 
    def showdetails(self):
        if not hasattr(self, 'current_user'):
            print("❌ No user logged in.")
            return
        print("\n--- YOUR DETAILS ---")
        for key, value in self.current_user.items():
            if key == 'pin':
                print(f"PIN: ****")
            elif key == 'borrowed_books':
                print(f"Borrowed Books: {value if value else 'None'}")
            else:
                print(f"{key}: {value}")
 
    def borrowbooks(self):
        # FIX: use logged in user's account type instead of asking again
        account_type = self.current_user.get('account_type', 'basic')
        available_books = self.books[account_type]
 
        print(f"\n📚 {account_type.upper()} BOOKS")
        print("-" * 30)
 
        has_available = False
        for book in available_books:
            if book['available']:
                print(f"  ✅ {book['id']}: {book['title']}")
                has_available = True
            else:
                print(f"  ❌ {book['id']}: {book['title']} (Borrowed)")
 
        if not has_available:
            print("❌ No books available to borrow.")
            return
 
        book_id = input("\nEnter Book ID to borrow: ").strip().upper()
 
        # FIX: actually mark book as borrowed
        for book in available_books:
            if book['id'] == book_id:
                if not book['available']:
                    print("❌ This book is already borrowed.")
                    return
                book['available'] = False
                self.current_user.setdefault('borrowed_books', []).append(book['title'])
                Librarysystem.update_data()
                print(f"✅ You have successfully borrowed '{book['title']}'!")
                return
 
        print("❌ Invalid Book ID.")
 
    def returnbooks(self):
        # FIX: use self.current_user directly, no need to login again
        borrowed = self.current_user.get('borrowed_books', [])
 
        if not borrowed:
            print("❌ You have no borrowed books to return.")
            return
 
        print("\n📚 YOUR BORROWED BOOKS")
        print("-" * 30)
        for i, title in enumerate(borrowed, 1):
            print(f"  {i}. {title}")
 
        title = input("\nEnter book title to return: ").strip()
 
        # FIX: actually mark book as available again
        found = False
        for category in ['basic', 'premium']:
            for book in self.books[category]:
                if book['title'].lower() == title.lower():
                    if title in borrowed:
                        book['available'] = True
                        self.current_user['borrowed_books'].remove(title)
                        Librarysystem.update_data()
                        print(f"✅ '{title}' returned successfully!")
                        found = True
                    else:
                        print("❌ You didn't borrow this book.")
                        found = True
                    break
            if found:
                break
 
        if not found:
            print("❌ Book not found in the system.")
 
    def updatedetails(self):
        if not hasattr(self, 'current_user'):
            print("❌ Please login first.")
            return
 
        print("Update your details (press Enter to keep current value):")
        print("Note: Account number, age, and account type cannot be changed.")
 
        new_name = input(f"New name [{self.current_user['name']}]: ").strip()
        new_email = input(f"New email [{self.current_user['email']}]: ").strip()
        new_pin = input(f"New 4-digit PIN [****]: ").strip()
 
        # Only update if something was entered
        if new_name:
            self.current_user['name'] = new_name
        if new_email:
            self.current_user['email'] = new_email
        if new_pin:
            if len(new_pin) == 4 and new_pin.isdigit():
                self.current_user['pin'] = int(new_pin)
            else:
                print("❌ PIN must be exactly 4 digits. PIN not updated.")
 
        Librarysystem.update_data()
        print("✅ Details updated successfully!")
 
    def cancel(self):
        if not hasattr(self, 'current_user'):
            print("❌ Please login first.")
            return
 
        print(f"⚠️  You are about to delete account for {self.current_user['name']}.")
        check = input("Are you sure? Type 'yes' to delete: ").strip()
 
        if check.lower() == 'yes':
            Librarysystem.storage.remove(self.current_user)
            Librarysystem.update_data()
            del self.current_user
            print("✅ Account deleted successfully.")
        else:
            print("Deletion cancelled.")
 
 
def main():
    Librarysystem.load_data()
    a = Librarysystem()
 
    while True:
        print("=" * 50)
        print("               SABIN LIBRARY ")
        print("=" * 50)
        print("1. Create your Library Membership account")
        print("2. Login into your Membership account")
        print("3. Show Membership details")
        print("4. Borrow books")
        print("5. Return books")
        print("6. Update the details")
        print("7. Cancel Membership")
        print("8. Exit")
        print("=" * 50)
 
        try:
            check = int(input("Enter your choice (1-8): "))
 
            if check == 1:
                a.createaccount()
            elif check == 2:
                a.login()
            elif check == 3:
                a.showdetails()
            elif check == 4:
                a.borrowbooks()
            elif check == 5:
                a.returnbooks()
            elif check == 6:
                a.updatedetails()
            elif check == 7:
                a.cancel()
            elif check == 8:
                print("\n✅ Thank you for using Sabin Library!")
                print("👋 Goodbye!")
                break
            else:
                print("❌ Please enter a valid number (1-8)")
 
        except ValueError:
            print("❌ Please enter a number, not text!")
        except Exception as err:
            print(f"❌ Error has occurred: {err}")
 
 
if __name__ == "__main__":
    main()
