import streamlit as st
from pydantic import BaseModel

# ---------- Initialize session state ----------
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False

if "books_db" not in st.session_state:
    st.session_state.books_db = {}

if "orders" not in st.session_state:
    st.session_state.orders = []

# ---------- BOOK MODEL ----------
class Book(BaseModel):
    book_id: int
    name: str
    author: str
    copies: int

# ---------- LOGIN PAGE ----------
st.title("📚 Bookstore Management App")

if not st.session_state.logged_in:
    st.header("🔑 Login")
    username = st.text_input("Username")
    password = st.text_input("Password", type="password")
    if st.button("Login"):
        if username == "library" and password == "1234":
            st.session_state.logged_in = True
            st.success("✔ Login Successful!")
            st.experimental_rerun()  # Refresh app after login
        else:
            st.error("❌ Invalid username or password")

else:
    # ---------- MAIN MENU ----------
    menu = st.sidebar.selectbox(
        "Menu",
        ["Add Book", "Update Book", "Get Book", "Place Order", "Checkout", "Logout"]
    )

    # ---------- LOGOUT ----------
    if menu == "Logout":
        st.session_state.logged_in = False
        st.experimental_rerun()

    # ---------- ADD BOOK ----------
    elif menu == "Add Book":
        st.header("➕ Add Book")
        book_id = st.number_input("Book ID", min_value=1, key="add_book_id")
        name = st.text_input("Book Name", key="add_name")
        author = st.text_input("Author", key="add_author")
        copies = st.number_input("No. of Copies", min_value=1, key="add_copies")

        if st.button("Add Book"):
            if book_id in st.session_state.books_db:
                st.error("❌ Book ID already exists!")
            else:
                book = Book(
                    book_id=book_id,
                    name=name,
                    author=author,
                    copies=copies
                )
                st.session_state.books_db[book_id] = book
                st.success(f"✔ Book ID {book_id} added successfully")

    # ---------- UPDATE BOOK ----------
    elif menu == "Update Book":
        st.header("📝 Update Book")
        update_id = st.number_input("Enter Book ID to Update", min_value=1, key="update_book_id")

        if update_id in st.session_state.books_db:
            book = st.session_state.books_db[update_id]
            name = st.text_input("Book Name", value=book.name, key="update_name")
            author = st.text_input("Author", value=book.author, key="update_author")
            copies = st.number_input("Copies", value=book.copies, min_value=1, key="update_copies")

            if st.button("Update Book"):
                updated_book = Book(
                    book_id=update_id,
                    name=name,
                    author=author,
                    copies=copies
                )
                st.session_state.books_db[update_id] = updated_book
                st.success(f"✔ Book ID {update_id} updated successfully")
        else:
            st.info("❌ Book ID not found. Please add the book first.")

    # ---------- GET BOOK ----------
    elif menu == "Get Book":
        st.header("📖 Get Book Details")
        book_id = st.number_input("Enter Book ID", min_value=1, key="get_book_id")

        if st.button("Get Book"):
            if book_id not in st.session_state.books_db:
                st.error("❌ Book not found")
            else:
                book = st.session_state.books_db[book_id]
                st.json(book.dict())

    # ---------- PLACE ORDER ----------
    elif menu == "Place Order":
        st.header("🛒 Place Order")
        book_id = st.number_input("Enter Book ID", min_value=1, key="order_book_id")
        qty = st.number_input("Quantity", min_value=1, key="order_qty")

        if st.button("Place Order"):
            if book_id not in st.session_state.books_db:
                st.error("❌ Book not found")
            else:
                book = st.session_state.books_db[book_id]
                if book.copies < qty:
                    st.error("❌ Not enough copies available!")
                else:
                    book.copies -= qty
                    st.session_state.orders.append({"book_id": book_id, "qty": qty})
                    st.success(f"✔ Order placed successfully for Book ID {book_id}")

    # ---------- CHECKOUT ----------
    elif menu == "Checkout":
        st.header("💳 Checkout Summary")
        st.write("### Total Orders:", len(st.session_state.orders))
        st.json(st.session_state.orders)

