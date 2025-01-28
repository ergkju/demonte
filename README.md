import customtkinter as ctk
import sqlite3
import hashlib
from datetime import datetime



class CustomMessageBox:
    def __init__(self, title, message, type="info"):
        self.dialog = ctk.CTkToplevel()
        self.dialog.title(title)
        self.dialog.geometry("400x200")

        # Center the dialog
        screen_width = self.dialog.winfo_screenwidth()
        screen_height = self.dialog.winfo_screenheight()
        x = (screen_width - 400) // 2
        y = (screen_height - 200) // 2
        self.dialog.geometry(f"400x200+{x}+{y}")

        # Make it modal
        self.dialog.transient()
        self.dialog.grab_set()

        # Color based on type
        if type == "error":
            title_color = "red"
        elif type == "success":
            title_color = "#00008B"  # Dark blue
        else:
            title_color = "#1f538d"  # Default blue

        # Title
        title_label = ctk.CTkLabel(
            self.dialog,
            text=title,
            font=("Impact", 20),
            text_color=title_color
        )
        title_label.pack(pady=20)

        # Message
        message_label = ctk.CTkLabel(
            self.dialog,
            text=message,
            font=("Impact", 14)
        )
        message_label.pack(pady=20)

        # OK Button
        ok_button = ctk.CTkButton(
            self.dialog,
            text="OK",
            command=self.dialog.destroy,
            font=("Impact", 14),
            fg_color=title_color,
            width=100
        )
        ok_button.pack(pady=20)


def show_error(title, message):
    CustomMessageBox(title, message, type="error")


def show_info(title, message):
    CustomMessageBox(title, message, type="success")


# Database Setup
conn = sqlite3.connect("users.db")
cursor = conn.cursor()

# Create tables if they don't exist
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    username TEXT PRIMARY KEY,
    password TEXT
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS application_cis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- Auto-incrementing primary key
    app_name TEXT,
    environment TEXT,
    version TEXT,
    owner TEXT,
    status TEXT,
    dependencies TEXT,
    ci_id TEXT UNIQUE,  -- Unique CI ID (can be auto-generated)
    app_license TEXT,
    description TEXT
)
""")
print("Application CI table created or already exists.")

cursor.execute("""
CREATE TABLE IF NOT EXISTS database_cis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- Auto-incrementing primary key
    name TEXT,
    type TEXT,
    version TEXT,
    environment TEXT,
    status TEXT,
    schema TEXT,
    owner TEXT,
    backup_details TEXT,
    dependencies TEXT,
    ci_id TEXT UNIQUE,  -- Unique CI ID (can be auto-generated)
    description TEXT
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS api_cis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- Auto-incrementing primary key
    name TEXT,
    version TEXT,
    type TEXT,
    protocol TEXT,
    endpoint_url TEXT,
    status TEXT,
    vendor TEXT,
    dependencies TEXT,
    environment TEXT,
    authentication_method TEXT,
    description TEXT,
    ci_id TEXT UNIQUE  -- Unique CI ID (can be auto-generated)
)
""")
print("API CI table created or already exists.")

# Create the vm_cis table if it doesn't exist
cursor.execute("""
CREATE TABLE IF NOT EXISTS vm_cis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    host_machine TEXT,
    version TEXT,
    memory_size TEXT,
    status TEXT,
    vm_template TEXT,
    snapshots TEXT,
    disk TEXT,
    creation_date TEXT,
    dependencies TEXT,
    description TEXT,
    ci_id TEXT UNIQUE
)
""")
conn.commit()




def generate_unique_ci_id(table_name):
    # Fetch the last inserted ID from the table
    cursor.execute(f"SELECT MAX(id) FROM {table_name}")
    last_id = cursor.fetchone()[0]

    # If no records exist, start with 1
    if last_id is None:
        last_id = 0

    # Generate a unique CI ID (e.g., APP001, DB002)
    prefix = "APP" if table_name == "application_cis" else "DB"
    new_id = last_id + 1
    unique_ci_id = f"{prefix}{new_id:03d}"  # Format as 3-digit number (e.g., APP001)

    return unique_ci_id




def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()


def sign_up():
    username = entry_username.get()
    password = entry_password.get()

    if not username or not password:
        show_error("Error", "Both fields are required!")
        return

    # Hash the password
    hashed_password = hash_password(password)

    try:
        cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, hashed_password))
        conn.commit()
        show_info("Success", "Account created successfully! Please login.")
    except sqlite3.IntegrityError:
        show_error("Error", "Username already exists!")


def login():
    username = entry_username.get()
    password = entry_password.get()

    if not username or not password:
        show_error("Error", "Both fields are required!")
        return

    # Hash the input password
    hashed_password = hash_password(password)

    cursor.execute("SELECT * FROM users WHERE username = ? AND password = ?", (username, hashed_password))
    user = cursor.fetchone()

    if user:
        show_info("Login Success", f"Welcome, {username}!")
        # Show dashboard and hide login frame
        login_frame.pack_forget()
        dashboard_frame.pack(expand=True, padx=50, pady=50, fill="both")
        # Update the welcome message with the logged-in username
        label_welcome.configure(text=f"Welcome, {username}!")
    else:
        show_error("Login Failed", "Invalid username or password!")


def exit_app():
    conn.close()  # Close the database connection
    root.destroy()  # Closes the application


def go_to_login():
    home_frame.pack_forget()  # Hide the home page frame
    login_frame.pack(expand=True, padx=50, pady=50, fill="both")  # Show login page frame


def logout():
    # Hide the dashboard frame and show login frame
    dashboard_frame.pack_forget()
    login_frame.pack(expand=True, padx=50, pady=50, fill="both")


def go_to_add_ci():
    dashboard_frame.pack_forget()
    add_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

'''def go_to_dashboard():
    # Hide all other frames
    add_ci_frame.pack_forget()  # Hide the Add CI frame
    view_ci_frame.pack_forget()  # Hide the View CI frame
    dashboard_frame.pack(expand=True, padx=50, pady=50, fill="both")  # Show the Dashboard frame'''

def go_to_dashboard():
    # Hide all other frames
    delete_ci_frame.pack_forget()
    add_ci_frame.pack_forget()
    view_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    api_ci_frame.pack_forget()
    vm_ci_frame.pack_forget()
    os_ci_frame.pack_forget()

    # Show the Dashboard frame
    dashboard_frame.pack(expand=True, padx=50, pady=50, fill="both")


# Function to go to the Application CI page
def go_to_application_ci():
    # Hide all other frames
    home_frame.pack_forget()
    login_frame.pack_forget()
    dashboard_frame.pack_forget()
    add_ci_frame.pack_forget()

    # Show the Application CI frame
    application_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")


def go_to_database_ci():
    # Hide all other frames
    print("Navigating to Database CI page")
    home_frame.pack_forget()
    login_frame.pack_forget()
    dashboard_frame.pack_forget()
    add_ci_frame.pack_forget()
    application_ci_frame.pack_forget()

    # Show the Database CI frame
    database_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Function to go to the View CI page
def go_to_view_ci():
    # Hide all other frames
    home_frame.pack_forget()
    login_frame.pack_forget()
    dashboard_frame.pack_forget()
    add_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()

    # Show the View CI frame
    view_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")


def save_application_ci():
    # Retrieve values from the entry fields
    app_name = entries["app_name"].get()
    environment = entries["environment"].get()
    version = entries["version"].get()
    owner = entries["owner"].get()
    status = entries["status"].get()
    dependencies = entries["dependencies"].get()
    app_license = entries["app_license"].get()
    description = text_description.get("1.0", "end-1c")

    # Validate that all required fields are filled
    if not app_name or not environment or not version or not owner or not status or not app_license:
        show_error("Error", "All fields are required except Description!")
        return

    # Generate a unique CI ID
    unique_ci_id = generate_unique_ci_id("application_cis")

    try:
        # Save the data to the database
        cursor.execute("""
        INSERT INTO application_cis (app_name, environment, version, owner, status, dependencies, ci_id, app_license, description)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (app_name, environment, version, owner, status, dependencies, unique_ci_id, app_license, description))
        conn.commit()  # Ensure changes are committed to the database
        print("Application CI saved successfully! CI ID:", unique_ci_id)
        show_info("Success", f"Application CI saved successfully! CI ID: {unique_ci_id}")
    except sqlite3.Error as e:
        print("Failed to save data:", e)
        show_error("Error", f"Failed to save data: {e}")

def save_database_ci():
    try:
        # Retrieve values from the entry fields
        name = entries_db["name"].get()
        type = entries_db["type"].get()
        version = entries_db["version"].get()
        environment = entries_db["environment"].get()
        status = entries_db["status"].get()
        schema = entries_db["schema"].get()
        owner = entries_db["owner"].get()
        backup_details = entries_db["backup_details"].get()
        dependencies = entries_db["dependencies"].get()
        description = text_description_db.get("1.0", "end-1c")

        # Validate that all required fields are filled
        if not name or not type or not version or not environment or not status or not schema or not owner:
            show_error("Error", "All fields are required except Description!")
            return

        # Generate a unique CI ID
        unique_ci_id = generate_unique_ci_id("database_cis")

        # Save the data to the database
        cursor.execute("""
        INSERT INTO database_cis (name, type, version, environment, status, schema, owner, backup_details, dependencies, ci_id, description)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (name, type, version, environment, status, schema, owner, backup_details, dependencies, unique_ci_id, description))
        conn.commit()  # Ensure changes are committed to the database
        print("Database CI saved successfully! CI ID:", unique_ci_id)
        show_info("Success", f"Database CI saved successfully! CI ID: {unique_ci_id}")
    except KeyError as e:
        print(f"KeyError: {e}. Ensure the key exists in entries_db.")
        show_error("Error", f"Missing field: {e}")
    except sqlite3.Error as e:
        print("Failed to save data:", e)
        show_error("Error", f"Failed to save data: {e}")

def save_os_ci():
    # Retrieve values from the entry fields
    name = entries_os["name"].get()
    version = entries_os["version"].get()
    edition = entries_os["edition"].get()
    architecture = entries_os["architecture"].get()
    kernel_version = entries_os["kernel_version"].get()
    license = entries_os["license"].get()
    ip_address = entries_os["ip_address"].get()
    status = entries_os["status"].get()
    owner = entries_os["owner"].get()
    patch_level = entries_os["patch_level"].get()
    installation_date = entries_os["installation_date"].get()
    description = text_description_os.get("1.0", "end-1c")  # Get text from the description textbox

    # Validate that all required fields are filled
    if not name or not version or not edition or not architecture or not kernel_version or not license or not status or not owner:
        show_error("Error", "All fields are required except Description!")
        return

    # Generate a unique CI ID
    unique_ci_id = generate_unique_ci_id("os_cis")

    try:
        # Save the data to the database
        cursor.execute("""
        INSERT INTO os_cis (
            name, version, edition, architecture, kernel_version, license, ip_address, status, owner, patch_level, installation_date, ci_id, description
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            name, version, edition, architecture, kernel_version, license, ip_address, status, owner, patch_level, installation_date, unique_ci_id, description
        ))
        conn.commit()  # Commit the transaction
        show_info("Success", f"OS CI saved successfully! CI ID: {unique_ci_id}")
    except sqlite3.Error as e:
        print("Failed to save data:", e)
        show_error("Error", f"Failed to save data: {e}")

# Save Functionality
def save_api_ci():
    # Retrieve values from the entry fields
    name = entries_api["name"].get()
    version = entries_api["version"].get()
    type = entries_api["type"].get()
    protocol = entries_api["protocol"].get()
    endpoint_url = entries_api["endpoint_url"].get()
    status = entries_api["status"].get()
    vendor = entries_api["vendor"].get()
    dependencies = entries_api["dependencies"].get()
    environment = entries_api["environment"].get()
    authentication_method = entries_api["authentication_method"].get()
    description = text_description_api.get("1.0", "end-1c")

    # Validate that all required fields are filled
    if not name or not version or not type or not protocol or not endpoint_url or not status or not vendor:
        show_error("Error", "All fields are required except Description!")
        return

    # Generate a unique CI ID
    unique_ci_id = generate_unique_ci_id("api_cis")

    try:
        # Save the data to the database
        cursor.execute("""
        INSERT INTO api_cis (
            name, version, type, protocol, endpoint_url, status, vendor, dependencies, environment, authentication_method, description, ci_id
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            name, version, type, protocol, endpoint_url, status, vendor, dependencies, environment, authentication_method, description, unique_ci_id
        ))
        conn.commit()  # Commit the transaction
        show_info("Success", f"API's CI saved successfully! CI ID: {unique_ci_id}")
    except sqlite3.Error as e:
        print("Failed to save data:", e)
        show_error("Error", f"Failed to save data: {e}")

# Save Functionality for Virtual Machine CI
def save_vm_ci():
    # Retrieve values from the entry fields
    name = entries_vm["name"].get()
    host_machine = entries_vm["host_machine"].get()
    version = entries_vm["version"].get()
    memory_size = entries_vm["memory_size"].get()
    status = entries_vm["status"].get()
    vm_template = entries_vm["vm_template"].get()
    snapshots = entries_vm["snapshots"].get()
    disk = entries_vm["disk"].get()
    creation_date = entries_vm["creation_date"].get()
    dependencies = entries_vm["dependencies"].get()
    description = text_description_vm.get("1.0", "end-1c")

    # Validate that all required fields are filled
    if not name or not host_machine or not version or not memory_size or not status:
        show_error("Error", "All fields are required except Description!")
        return

    # Generate a unique CI ID
    unique_ci_id = generate_unique_ci_id("vm_cis")

    try:
        # Save the data to the database
        cursor.execute("""
        INSERT INTO vm_cis (
            name, host_machine, version, memory_size, status, vm_template, snapshots, disk, creation_date, dependencies, description, ci_id
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            name, host_machine, version, memory_size, status, vm_template, snapshots, disk, creation_date, dependencies, description, unique_ci_id
        ))
        conn.commit()  # Commit the transaction
        show_info("Success", f"Virtual Machine CI saved successfully! CI ID: {unique_ci_id}")
    except sqlite3.Error as e:
        print("Failed to save data:", e)
        show_error("Error", f"Failed to save data: {e}")

# App Setup
ctk.set_appearance_mode("Dark")  # Options: "System", "Dark", "Light"
ctk.set_default_color_theme("dark-blue")  # Setting the default theme color to ink blue

# Create the main application frame
root = ctk.CTk()
root.title("CMDB Software")
root.state('zoomed')  # Maximizes the window when launched

# Home Page Frame
home_frame = ctk.CTkFrame(root, width=600, height=500, border_width=5, border_color="#00008B")
home_frame.pack(expand=True, padx=50, pady=50, fill="both")

# CMDB Software Label at the Top
label_title = ctk.CTkLabel(home_frame, text="CMDB SOFTWARE", font=("Impact", 90))
label_title.pack(pady=(120, 30))

# Buttons (Let's Start and Exit)
button_start = ctk.CTkButton(home_frame, text="Admin", command=go_to_login, width=400, font=("Impact", 15))
button_start.pack(pady=20)

# User Button
button_user = ctk.CTkButton(home_frame, text=" User ", command=go_to_login, width=400, font=("Impact", 15),
                            fg_color="green")
button_user.pack(pady=10)

button_exit = ctk.CTkButton(home_frame, text="Exit", command=exit_app, width=350, font=("Impact", 15), fg_color="red")
button_exit.pack(pady=10)

# Login Page Frame (hidden initially)
login_frame = ctk.CTkFrame(root, width=600, height=500, border_width=3, border_color="#FF0000", fg_color="transparent")
login_frame.pack_forget()  # Initially hidden

# Title Label for Login Page
label_title_login = ctk.CTkLabel(login_frame, text="CMDB Software", font=("Impact", 80))
label_title_login.pack(pady=40)

# Username Entry
entry_username = ctk.CTkEntry(login_frame, placeholder_text="Enter Username", width=400, height=40, font=("Impact", 15))
entry_username.pack(pady=10)

# Password Entry
entry_password = ctk.CTkEntry(login_frame, placeholder_text="Enter Password", show="*", width=400, height=40,
                              font=("Impact", 15))
entry_password.pack(pady=20)

# Buttons (Login, Sign Up, Exit)
button_login = ctk.CTkButton(login_frame, text="Login", command=login, width=250, font=("Impact", 15))
button_login.pack(pady=10)

button_signup = ctk.CTkButton(login_frame, text="Sign Up", command=sign_up, width=250, font=("Impact", 15))
button_signup.pack(pady=20)

#button_exit = ctk.CTkButton(login_frame, text="Exit", command=exit_app, width=200, font=("Impact", 15), fg_color="red")
#button_exit.pack(pady=10)

# Back Button (Redirects to Home Page)
button_back = ctk.CTkButton(
    login_frame,
    text="Back",
    command=lambda: go_to_home(),  # Redirect to home page
    width=200,
    font=("Impact", 15),
    fg_color="red"
)
button_back.pack(pady=10)

# Function to go back to the home page
def go_to_home():
    login_frame.pack_forget()  # Hide the login page
    home_frame.pack(expand=True, padx=50, pady=50, fill="both")  # Show the home page

# Dashboard Frame
dashboard_frame = ctk.CTkFrame(root, width=600, height=500, border_width=5, border_color="#00008B")
dashboard_frame.pack_forget()

# Dashboard Title (Maximized size and moved to the left top corner)
label_title_dashboard = ctk.CTkLabel(dashboard_frame, text="CMDB Dashboard", font=("Impact", 60))  # Increased font size
label_title_dashboard.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the CMDB Dashboard title)
horizontal_line = ctk.CTkFrame(dashboard_frame, height=6, fg_color="#00008B")
horizontal_line.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Welcome Message Box
welcome_box = ctk.CTkFrame(
    dashboard_frame,
    border_width=2,  # Border thickness
    border_color="#00008B",  # Border color
    fg_color="#1f538d"  # Background color of the box
)
welcome_box.pack(pady=(150, 10))  # Padding above and below the box

# Welcome Label (Inside the box)
label_welcome = ctk.CTkLabel(
    welcome_box,
    text="Welcome, Admin!",  # Replace "Admin" with the logged-in username
    font=("Impact", 30),
    fg_color="#1f538d",  # Background color of the label (matches the box)
    text_color="white"  # Text color
)
label_welcome.pack(padx=20, pady=20)  # Padding inside the box


# Date and Time Display
def update_time():
    now = datetime.now()
    date_time = now.strftime("Date: %d-%m-%Y\nTime: %I:%M:%S %p")
    label_date_time.configure(text=date_time)
    label_date_time.after(1000, update_time)


label_date_time = ctk.CTkLabel(dashboard_frame, text="", font=("Impact", 20))
label_date_time.pack(pady=(10, 30))
update_time()

# Scrollable Frame for Dashboard Content (Placed below the horizontal line at the left corner)
scrollable_frame = ctk.CTkScrollableFrame(dashboard_frame, width=200, fg_color="transparent",
                                          label_text="CMDB EX",
                                          label_font=("Impact", 20),
                                          label_fg_color="#1f538d",
                                          )
scrollable_frame.place(x=10, y=200)  # Use x=20 and y=200 to position it

# Buttons for Dashboard Features
buttons = [
    "Dependencies", "CI Types", "Filters", "Statistics", "Updates", "CI List", "Relationship"
]
for i, button_text in enumerate(buttons):
    button = ctk.CTkButton(
        scrollable_frame,
        text=button_text,
        font=("Impact", 15),
        fg_color="transparent",  # No specific color
        hover_color="#1f538d"  # Hover color for better UX
    )
    button.pack(pady=5)  # Place

# Dashboard Buttons Frame
buttons_frame = ctk.CTkFrame(dashboard_frame, fg_color="transparent")
buttons_frame.pack(pady=20)

# Add CI Button
button_add_ci = ctk.CTkButton(
    buttons_frame,
    text="Add CI",
    command=go_to_add_ci,
    width=200,
    font=("Impact", 15),
    hover_color="green",
    fg_color="#1f538d"
)
button_add_ci.grid(row=0, column=0, padx=10, pady=10)

# View CI Button
button_view_ci = ctk.CTkButton(
    buttons_frame,
    text="View CI",
    command=go_to_view_ci,
    width=200,
    font=("Impact", 15),
    hover_color="green",
    fg_color="#1f538d"
)
button_view_ci.grid(row=0, column=1, padx=10, pady=10)


def go_to_delete_ci():
    # Hide all other frames
    dashboard_frame.pack_forget()
    add_ci_frame.pack_forget()
    view_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    api_ci_frame.pack_forget()
    vm_ci_frame.pack_forget()
    os_ci_frame.pack_forget()

    # Show the Delete CI frame
    delete_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# delete CI button
button_delete_ci = ctk.CTkButton(
    buttons_frame,
    text="Delete CI",
    command=go_to_delete_ci,  # Redirect to the Delete CI page
    width=200,
    font=("Impact", 15),
    hover_color="green",
    fg_color="#1f538d"
)
button_delete_ci.grid(row=1, column=0, padx=10, pady=10)

def go_to_update_ci():
    # Hide all other frames
    dashboard_frame.pack_forget()
    add_ci_frame.pack_forget()
    view_ci_frame.pack_forget()
    delete_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    api_ci_frame.pack_forget()
    vm_ci_frame.pack_forget()
    os_ci_frame.pack_forget()

    # Show the Update CI frame
    update_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Update CI Button
button_update_ci = ctk.CTkButton(
    buttons_frame,
    text="Update CI",
    command=go_to_update_ci,  # Redirect to the Update CI page
    width=200,
    font=("Impact", 15),
    hover_color="green",
    fg_color="#1f538d"
)
button_update_ci.grid(row=1, column=1, padx=10, pady=10)

# Logout Button (Placed at the top-right corner)
button_logout = ctk.CTkButton(
    dashboard_frame,
    text="Logout",
    command=logout,
    width=150,  # Increased width
    height=60,  # Increased height
    font=("Impact", 18),  # Increased font size
    fg_color="red"
)
button_logout.place(relx=0.95, rely=0.07, anchor="ne")  # Positioned at the top-right corner

# Add CI Frame
add_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5,
                            border_color="#00008B")  # Increased frame size
add_ci_frame.pack_forget()

# Add CI Title (Maximized size and moved to the left top corner)
label_title_add_ci = ctk.CTkLabel(add_ci_frame, text="CONFIGURATION ITEM - ADD",
                                  font=("Impact", 60))  # Increased font size
label_title_add_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the Add CI title)
horizontal_line_add_ci = ctk.CTkFrame(add_ci_frame, height=6, fg_color="#00008B")
horizontal_line_add_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Frame to hold CI type buttons
ci_type_buttons_frame = ctk.CTkFrame(add_ci_frame, fg_color="transparent")
ci_type_buttons_frame.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame
ci_type_buttons_frame.pack_forget()  # Initially hidden

# Frame to hold Object type group buttons
buttons_frame = ctk.CTkFrame(add_ci_frame, fg_color="transparent")
buttons_frame.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame


def show_ci_type_buttons():
    # Hide the Object type group buttons frame
    buttons_frame.pack_forget()

    # Clear any existing buttons in the CI type buttons frame
    for widget in ci_type_buttons_frame.winfo_children():
        widget.destroy()

    # List of CI type buttons
    ci_type_buttons = ["Application", "Database", "OS", "API's", "Virtual Machine", "Software Licenses"]

    # Create and display the CI type buttons
    for i, button_text in enumerate(ci_type_buttons):
        if button_text == "Application":
            # Application button
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=go_to_application_ci  # Bind to go_to_application_ci
            )
        elif button_text == "Database":
            # Database button
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=go_to_database_ci  # Bind to go_to_database_ci
            )
        elif button_text == "OS":
            # OS button
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=go_to_os_ci  # Bind to go_to_os_ci
            )
        elif button_text == "API's":
            # API's button
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=go_to_api_ci  # Bind to go_to_api_ci
            )
        elif button_text == "Virtual Machine":
            # Virtual Machine button
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=go_to_vm_ci  # Bind to go_to_vm_ci
            )
        else:
            # Other buttons (e.g., Software Licenses)
            button = ctk.CTkButton(
                ci_type_buttons_frame,
                text=button_text,
                font=("Impact", 14),
                fg_color="#1f538d",  # Button color
                hover_color="#00008B",  # Hover color
                width=150,  # Button width
                height=40,  # Button height
                corner_radius=10,  # Rounded corners
                command=lambda bt=button_text: show_info("Info", f"{bt} functionality not implemented yet.")
            )

        # Arrange buttons in a grid
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)

    # Show the CI type buttons frame
    ci_type_buttons_frame.pack(expand=True, padx=300, pady=(0, 50), fill="both")

def show_object_type_buttons():
    # Hide the CI type buttons frame
    ci_type_buttons_frame.pack_forget()

    # Clear any existing buttons in the Object type group buttons frame
    for widget in buttons_frame.winfo_children():
        widget.destroy()

    # List of Object type group buttons
    button_labels = ["Software", "Hardware", "Network", "Documentation", "Information", "Cloud", "Facilities"]

    # Create and display the Object type group buttons
    for i, label in enumerate(button_labels):
        button = ctk.CTkButton(
            buttons_frame,
            text=label,
            font=("Impact", 16),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10  # Rounded corners
        )
        if label == "Software":
            button.configure(command=show_ci_type_buttons)  # Bind "Software" button to show CI type buttons
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)  # Arrange buttons in a grid (3 columns)

    # Show the Object type group buttons frame
    buttons_frame.pack(expand=True, padx=300, pady=(0, 50), fill="both")

# Blue Toolbar Box
toolbar_box = ctk.CTkFrame(
    add_ci_frame,
    border_width=1,  # Border thickness
    border_color="#00008B",  # Border color
    fg_color="#00008B",  # Background color of the box
    height=60,
    width=1200  # Set the height of the toolbar box (adjust as needed)
)
toolbar_box.pack(pady=110, fill="both") #Padding above and below the box, and stretch horizontally

# Buttons for the Toolbar (CI, Dashboard, Report, Object Type Group, Charts)
toolbar_buttons = ["Object type group", "Dashboard", "Dependencies", "Relationship"]


# Place buttons on the toolbar
for i, button_text in enumerate(toolbar_buttons):
    button = ctk.CTkButton(
        toolbar_box,
        text=button_text,
        font=("Impact", 14),
        fg_color="transparent",  # Transparent background
        hover_color="#1f538d",  # Hover color
        width=120,  # Button width
        height=30,  # Button height
        corner_radius=10
    )
    if button_text == "Object type group":
        button.configure(command=show_object_type_buttons)  # Bind the "Object type group" button to the function
    button.grid(row=0, column=i, padx=90, pady=5)  # Place buttons in a grid layout

# Back Button (Placed at the bottom left corner)
button_back = ctk.CTkButton(
    add_ci_frame,
    text="Back",
    command=lambda: go_to_dashboard(),  # Function to go back to the dashboard
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner

# Application CI Frame
application_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5,
                                    border_color="#00008B")  # Increased frame size
application_ci_frame.pack_forget()

# Application CI Title (Maximized size and moved to the left top corner)
label_title_application_ci = ctk.CTkLabel(application_ci_frame, text="APPLICATION CI - ADD", font=("Impact", 60))
label_title_application_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the Application CI title)
horizontal_line_application_ci = ctk.CTkFrame(application_ci_frame, height=6, fg_color="#00008B")
horizontal_line_application_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Entry Fields for Application CI (Left Side)
left_entry_fields = [
    ("App Name", 0.05, 0.30),  # Adjusted rely from 0.25 to 0.30
    ("Environment", 0.05, 0.40),  # Adjusted rely from 0.35 to 0.40
    ("Version", 0.05, 0.50),  # Adjusted rely from 0.45 to 0.50
    ("Owner", 0.05, 0.60),  # Adjusted rely from 0.55 to 0.60
    ("Dependencies", 0.05, 0.70),  # Adjusted rely from 0.65 to 0.70
]

# Entry Fields for Application CI (Right Side)
right_entry_fields = [  # Adjusted rely from 0.25 to 0.30
    ("App License", 0.55, 0.30),  # Adjusted rely from 0.35 to 0.40
    ("Status", 0.55, 0.40),  # Adjusted rely from 0.45 to 0.50
]

# Create entry fields (Left Side)
entries = {}
for field, relx, rely in left_entry_fields:
    label = ctk.CTkLabel(application_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(application_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries[field.lower().replace(" ", "_")] = entry

# Create entry fields (Right Side)
for field, relx, rely in right_entry_fields:
    label = ctk.CTkLabel(application_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(application_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries[field.lower().replace(" ", "_")] = entry  # Ensure correct key names

# Description Box (Right Side)
label_description = ctk.CTkLabel(application_ci_frame, text="Description", font=("Impact", 16))
label_description.place(relx=0.55, rely=0.70, anchor="w")

text_description = ctk.CTkTextbox(application_ci_frame, width=300, height=100, font=("Impact", 14))
text_description.place(relx=0.55 + 0.15, rely=0.70, anchor="w")


# Buttons (Save, Cancel, Add Custom Fields)
button_save = ctk.CTkButton(
    application_ci_frame,
    text="Save",
    command=save_application_ci,  # Bind the save_application_ci function
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="green"
)
button_save.place(relx=0.25, rely=0.95, anchor="sw")

# Function to clear Application CI fields
def clear_application_ci_fields():
    for entry in entries.values():
        entry.delete(0, "end")
    text_description.delete("1.0", "end")

button_cancel = ctk.CTkButton(
    application_ci_frame,
    text="Cancel",
    command=lambda: clear_application_ci_fields(),
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_cancel.place(relx=0.45, rely=0.95, anchor="sw")

button_add_custom_fields = ctk.CTkButton(
    application_ci_frame,
    text="Add Custom Fields",
    command=lambda: show_info("Info", "Custom fields functionality not implemented yet."),
    width=200,
    height=40,
    font=("Impact", 18),
    fg_color="#1f538d"
)
button_add_custom_fields.place(relx=0.70, rely=0.95, anchor="sw")


def go_to_add_ci():
    # Hide all other frames
    database_ci_frame.pack_forget()
    application_ci_frame.pack_forget()

    # Show the Add CI frame
    add_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Back Button (Placed at the bottom left corner)
button_back_application_ci = ctk.CTkButton(
    application_ci_frame,
    text="Back",
    command=lambda: go_to_add_ci(),  # Function to go back to the Add CI frame
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_application_ci.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner

# Database CI Frame
database_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5,
                                 border_color="#00008B")  # Increased frame size
database_ci_frame.pack_forget()

# API's CI Frame
api_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
api_ci_frame.pack_forget()

# API's CI Title
label_title_api_ci = ctk.CTkLabel(api_ci_frame, text="API's CI - ADD", font=("Impact", 60))
label_title_api_ci.place(relx=0.05, rely=0.05, anchor="nw")

# Horizontal Line (Below the API's CI title)
horizontal_line_api_ci = ctk.CTkFrame(api_ci_frame, height=6, fg_color="#00008B")
horizontal_line_api_ci.place(rely=0.20, relwidth=1.0, anchor="nw")

# Entry Fields for API's CI (Left Side)
left_entry_fields_api = [
    ("Name", 0.05, 0.30),
    ("Version", 0.05, 0.40),
    ("Type", 0.05, 0.50),
    ("Protocol", 0.05, 0.60),
    ("Endpoint URL", 0.05, 0.70),
]

# Entry Fields for API's CI (Right Side)
right_entry_fields_api = [
    ("Status", 0.55, 0.30),
    ("Vendor", 0.55, 0.40),
    ("Dependencies", 0.55, 0.50),
    ("Environment", 0.55, 0.60),
    ("Authentication Method", 0.55, 0.70),
]

# Create entry fields (Left Side)
entries_api = {}
for field, relx, rely in left_entry_fields_api:
    label = ctk.CTkLabel(api_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(api_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_api[field.lower().replace(" ", "_")] = entry

# Create entry fields (Right Side)
for field, relx, rely in right_entry_fields_api:
    label = ctk.CTkLabel(api_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(api_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_api[field.lower().replace(" ", "_")] = entry

# Description Box (Right Side)
label_description_api = ctk.CTkLabel(api_ci_frame, text="Description", font=("Impact", 16))
label_description_api.place(relx=0.55, rely=0.80, anchor="w")

text_description_api = ctk.CTkTextbox(api_ci_frame, width=300, height=50, font=("Impact", 14))
text_description_api.place(relx=0.55 + 0.15, rely=0.80, anchor="w")

# Cancel Functionality
def clear_api_ci_fields():
    for entry in entries_api.values():
        entry.delete(0, "end")
    text_description_api.delete("1.0", "end")

# Back Functionality
def go_to_add_ci():
    api_ci_frame.pack_forget()
    add_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Buttons (Save, Cancel, Back)
button_save_api = ctk.CTkButton(
    api_ci_frame,
    text="Save",
    command=save_api_ci,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="green"
)
button_save_api.place(relx=0.25, rely=0.95, anchor="sw")

button_cancel_api = ctk.CTkButton(
    api_ci_frame,
    text="Cancel",
    command=clear_api_ci_fields,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_cancel_api.place(relx=0.45, rely=0.95, anchor="sw")

button_back_api = ctk.CTkButton(
    api_ci_frame,
    text="Back",
    command=go_to_add_ci,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_back_api.place(relx=0.05, rely=0.95, anchor="sw")

# Virtual Machine CI Frame
vm_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
vm_ci_frame.pack_forget()

# Virtual Machine CI Title
label_title_vm_ci = ctk.CTkLabel(vm_ci_frame, text="VIRTUAL MACHINE CI - ADD", font=("Impact", 60))
label_title_vm_ci.place(relx=0.05, rely=0.05, anchor="nw")

# Horizontal Line (Below the Virtual Machine CI title)
horizontal_line_vm_ci = ctk.CTkFrame(vm_ci_frame, height=6, fg_color="#00008B")
horizontal_line_vm_ci.place(rely=0.20, relwidth=1.0, anchor="nw")

def go_to_vm_ci():
    # Hide all other frames
    add_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    api_ci_frame.pack_forget()
    os_ci_frame.pack_forget()

    # Show the Virtual Machine CI frame
    vm_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Entry Fields for Virtual Machine CI (Left Side)
left_entry_fields_vm = [
    ("Name", 0.05, 0.30),
    ("Host Machine", 0.05, 0.40),
    ("Version", 0.05, 0.50),
    ("Memory Size", 0.05, 0.60),
    ("Status", 0.05, 0.70),
]

# Entry Fields for Virtual Machine CI (Right Side)
right_entry_fields_vm = [
    ("VM Template", 0.55, 0.30),
    ("Snapshots", 0.55, 0.40),
    ("Disk", 0.55, 0.50),
    ("Creation Date", 0.55, 0.60),
    ("Dependencies", 0.55, 0.70),
]

# Create entry fields (Left Side)
entries_vm = {}
for field, relx, rely in left_entry_fields_vm:
    label = ctk.CTkLabel(vm_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(vm_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_vm[field.lower().replace(" ", "_")] = entry

# Create entry fields (Right Side)
for field, relx, rely in right_entry_fields_vm:
    label = ctk.CTkLabel(vm_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(vm_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_vm[field.lower().replace(" ", "_")] = entry

# Description Box (Right Side)
label_description_vm = ctk.CTkLabel(vm_ci_frame, text="Description", font=("Impact", 16))
label_description_vm.place(relx=0.55, rely=0.80, anchor="w")

text_description_vm = ctk.CTkTextbox(vm_ci_frame, width=300, height=50, font=("Impact", 14))
text_description_vm.place(relx=0.55 + 0.15, rely=0.80, anchor="w")

# Cancel Functionality for Virtual Machine CI
def clear_vm_ci_fields():
    for entry in entries_vm.values():
        entry.delete(0, "end")
    text_description_vm.delete("1.0", "end")

# Back Functionality for Virtual Machine CI
def go_to_add_ci_from_vm():
    vm_ci_frame.pack_forget()
    add_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# Buttons (Save, Cancel, Back)
button_save_vm = ctk.CTkButton(
    vm_ci_frame,
    text="Save",
    command=save_vm_ci,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="green"
)
button_save_vm.place(relx=0.25, rely=0.95, anchor="sw")

button_cancel_vm = ctk.CTkButton(
    vm_ci_frame,
    text="Cancel",
    command=clear_vm_ci_fields,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_cancel_vm.place(relx=0.45, rely=0.95, anchor="sw")

button_back_vm = ctk.CTkButton(
    vm_ci_frame,
    text="Back",
    command=go_to_add_ci_from_vm,
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_back_vm.place(relx=0.05, rely=0.95, anchor="sw")


# Delete CI Frame
delete_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
delete_ci_frame.pack_forget()

# Delete CI Title (Maximized size and moved to the left top corner)
label_title_delete_ci = ctk.CTkLabel(delete_ci_frame, text="CONFIGURATION ITEM - DELETE", font=("Impact", 60))
label_title_delete_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the Delete CI title)
horizontal_line_delete_ci = ctk.CTkFrame(delete_ci_frame, height=6, fg_color="#00008B")
horizontal_line_delete_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Search Bar and Search Button
search_entry_delete_ci = ctk.CTkEntry(
    delete_ci_frame,
    placeholder_text="Search CI...",
    width=400,
    height=40,
    font=("Impact", 14)
)
search_entry_delete_ci.place(relx=0.3, rely=0.25, anchor="nw")  # Positioned below the horizontal line

search_button_delete_ci = ctk.CTkButton(
    delete_ci_frame,
    text="Search",
    font=("Impact", 14),
    fg_color="#1f538d",
    hover_color="#00008B",
    width=100,
    height=40,
    corner_radius=10,
    command=lambda: search_ci_delete(search_entry_delete_ci.get())  # Bind the search function
)
search_button_delete_ci.place(relx=0.6, rely=0.25, anchor="nw")  # Positioned next to the search bar

clear_button_delete_ci = ctk.CTkButton(
    delete_ci_frame,
    text="Clear",
    font=("Impact", 14),
    fg_color="red",
    hover_color="#8B0000",
    width=100,
    height=40,
    corner_radius=10,
    command=lambda: clear_search_delete_ci()  # Bind the clear function
)
clear_button_delete_ci.place(relx=0.7, rely=0.25, anchor="nw")  # Positioned next to the search button

# Delete Button
delete_button_delete_ci = ctk.CTkButton(
    delete_ci_frame,
    text="Delete",
    font=("Impact", 14),
    fg_color="red",
    hover_color="#8B0000",
    width=100,
    height=40,
    corner_radius=10,
    command=lambda: delete_ci(search_entry_delete_ci.get())  # Bind the delete function
)
delete_button_delete_ci.place(relx=0.8, rely=0.25, anchor="nw")  # Positioned next to the Clear button

# Scrollable Frame to hold CI type buttons in Delete CI page
ci_type_buttons_scrollable_frame_delete_ci = ctk.CTkScrollableFrame(
    delete_ci_frame,
    fg_color="transparent",
    width=200,  # Adjust width as needed
    height=300  # Adjust height to fit within the frame
)
ci_type_buttons_scrollable_frame_delete_ci.place(relx=0.05, rely=0.25, anchor="nw")  # Positioned below the horizontal line

# List of CI type buttons
ci_type_buttons_delete_ci = ["Application", "Database", "OS", "API's", "Virtual Machine", "Software Licenses"]

# Create and display the CI type buttons
for i, button_text in enumerate(ci_type_buttons_delete_ci):
    if button_text == "Application":
        # Application button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda: display_application_table_delete_ci()  # Bind to display_application_table_delete_ci
        )
    elif button_text == "Database":
        # Database button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda: display_database_table_delete_ci()  # Bind to display_database_table_delete_ci
        )
    elif button_text == "OS":
        # OS button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda: display_os_table_delete_ci()  # Bind to display_os_table_delete_ci
        )
    elif button_text == "API's":
        # API's button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda: display_api_table_delete_ci()  # Bind to display_api_table_delete_ci
        )
    elif button_text == "Virtual Machine":
        # Virtual Machine button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda: display_vm_table_delete_ci()  # Bind to display_vm_table_delete_ci
        )
    else:
        # Other buttons (e.g., Software Licenses)
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame_delete_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda bt=button_text: show_info("Info", f"{bt} functionality not implemented yet.")
        )

    # Arrange buttons vertically (one below the other)
    button.grid(row=i, column=0, padx=10, pady=10)

# Vertical Line (Right after the scrollable frame)
vertical_line_delete_ci = ctk.CTkFrame(
    delete_ci_frame,
    width=4,  # Line width
    height=600,  # Match the height of the scrollable frame
    fg_color="#00008B"  # Line color
)
vertical_line_delete_ci.place(relx=0.25, rely=0.20, anchor="nw")  # Positioned right after the scrollable frame

# Function to display Application CI table in Delete CI page
def display_application_table_delete_ci():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT app_name, environment, version, owner, status, dependencies, ci_id, app_license, description
        FROM application_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Application Data:", data)

        # Define column headers
        headers = ["App Name", "Environment", "Version", "Owner", "Status", "Dependencies", "CI ID", "App License", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching application data:", e)

# Function to display Database CI table in Delete CI page
def display_database_table_delete_ci():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, type, version, environment, status, schema, owner, backup_details, dependencies, ci_id, description
        FROM database_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Database Data:", data)

        # Define column headers
        headers = ["Name", "Type", "Version", "Environment", "Status", "Schema", "Owner", "Backup Details", "Dependencies", "CI ID", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching database data:", e)

# Function to display OS CI table in Delete CI page
def display_os_table_delete_ci():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, version, edition, architecture, kernel_version, license, ip_address, status, owner, patch_level, installation_date, ci_id, description
        FROM os_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched OS Data:", data)

        # Define column headers
        headers = ["Name", "Version", "Edition", "Architecture", "Kernel Version", "License", "IP Address", "Status", "Owner", "Patch Level", "Installation Date", "CI ID", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching OS data:", e)

# Function to display API CI table in Delete CI page
def display_api_table_delete_ci():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, version, type, protocol, endpoint_url, status, vendor, dependencies, environment, authentication_method, description, ci_id
        FROM api_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched API Data:", data)

        # Define column headers
        headers = ["Name", "Version", "Type", "Protocol", "Endpoint URL", "Status", "Vendor", "Dependencies", "Environment", "Authentication Method", "Description", "CI ID"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching API data:", e)

# Function to display Virtual Machine CI table in Delete CI page
def display_vm_table_delete_ci():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, host_machine, version, memory_size, status, vm_template, snapshots, disk, creation_date, dependencies, description, ci_id
        FROM vm_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Virtual Machine Data:", data)

        # Define column headers
        headers = ["Name", "Host Machine", "Version", "Memory Size", "Status", "VM Template", "Snapshots", "Disk", "Creation Date", "Dependencies", "Description", "CI ID"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching Virtual Machine data:", e)

# Function to search CI in Delete CI page
def search_ci_delete(search_term):
    try:
        print(f"Searching for CI Name: {search_term}")  # Debugging: Print the search term

        # Ensure the database connection is open
        if not conn:
            print("Database connection is not open.")
            return

        # Query to search across all CI tables using only the CI Name
        query = """
        SELECT 'Application' AS ci_type, app_name AS name
        FROM application_cis
        WHERE app_name LIKE ?
        UNION ALL
        SELECT 'Database' AS ci_type, name
        FROM database_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'API' AS ci_type, name
        FROM api_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'OS' AS ci_type, name
        FROM os_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'VM' AS ci_type, name
        FROM vm_cis
        WHERE name LIKE ?
        """

        # Execute the query with the search term (only searching by CI Name)
        cursor.execute(query, (
            f"%{search_term}%",  # Application
            f"%{search_term}%",  # Database
            f"%{search_term}%",  # API
            f"%{search_term}%",  # OS
            f"%{search_term}%"   # VM
        ))
        data = cursor.fetchall()

        print(f"Fetched Data: {data}")  # Debugging: Print the fetched data

        if not data:
            print("No matching data found.")  # Debugging: Print if no data is found
            show_info("Info", "No matching data found.")
            return

        # Clear the existing table
        clear_table_frame_delete_ci()

        # Define column headers
        headers = ["CI Type", "Name"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_delete_ci, table_frame_delete_ci
        scrollable_table_frame_delete_ci = ctk.CTkScrollableFrame(
            delete_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_delete_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame_delete_ci = ctk.CTkFrame(scrollable_table_frame_delete_ci, fg_color="transparent")
        table_frame_delete_ci.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_delete_ci,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_delete_ci.update_idletasks()
    except sqlite3.Error as e:
        print("Error searching CI data:", e)
        show_error("Error", f"Failed to search data: {e}")

# Function to clear the table frame in Delete CI page
def clear_table_frame_delete_ci():
    # Destroy the existing table frame if it exists
    if 'scrollable_table_frame_delete_ci' in globals():
        scrollable_table_frame_delete_ci.destroy()

# Function to clear the search in Delete CI page
def clear_search_delete_ci():
    # Clear the search entry
    search_entry_delete_ci.delete(0, "end")

    # Clear the table frame
    clear_table_frame_delete_ci()


def delete_ci(search_term):
    try:
        if not search_term:
            show_error("Error", "Please enter a CI name to delete.")
            return

        # Confirm deletion with the user
        confirm = show_confirmation("Confirm Deletion", f"Are you sure you want to delete '{search_term}'?")
        if not confirm:
            return

        # Delete the CI from all tables
        tables = {
            "application_cis": "app_name",  # Column name for Application CI
            "database_cis": "name",         # Column name for Database CI
            "api_cis": "name",              # Column name for API CI
            "os_cis": "name",               # Column name for OS CI
            "vm_cis": "name"                # Column name for Virtual Machine CI
        }
        deleted = False

        for table, column in tables.items():
            cursor.execute(f"DELETE FROM {table} WHERE {column} LIKE ?", (f"%{search_term}%",))
            if cursor.rowcount > 0:
                deleted = True
                conn.commit()  # Commit the deletion

        if deleted:
            show_info("Success", f"'{search_term}' has been deleted successfully.")
            # Refresh the table after deletion
            search_ci_delete(search_term)  # Re-fetch and display the updated table
        else:
            show_error("Error", f"No CI found with the name '{search_term}'.")

    except sqlite3.Error as e:
        print("Error deleting CI:", e)
        show_error("Error", f"Failed to delete CI: {e}")

# Function to show a confirmation dialog
def show_confirmation(title, message):
    dialog = ctk.CTkToplevel()
    dialog.title(title)
    dialog.geometry("400x200")
    dialog.transient(root)  # Make the dialog a transient window of the root
    dialog.grab_set()  # Grab focus to the dialog

    # Center the dialog on the screen
    screen_width = dialog.winfo_screenwidth()
    screen_height = dialog.winfo_screenheight()
    x = (screen_width - 400) // 2
    y = (screen_height - 200) // 2
    dialog.geometry(f"400x200+{x}+{y}")

    # Message Label
    message_label = ctk.CTkLabel(dialog, text=message, font=("Impact", 16))
    message_label.pack(pady=20, padx=20, fill="both", expand=True)  # Center the message

    # Buttons Frame
    buttons_frame = ctk.CTkFrame(dialog, fg_color="transparent")
    buttons_frame.pack(pady=10)

    # Variable to store the user's choice
    confirm = False

    # Yes Button
    def on_yes():
        nonlocal confirm
        confirm = True
        dialog.destroy()

    button_yes = ctk.CTkButton(
        buttons_frame,
        text="Yes",
        command=on_yes,
        font=("Impact", 14),
        fg_color="green",
        width=100
    )
    button_yes.pack(side="left", padx=20, pady=10)

    # No Button
    def on_no():
        nonlocal confirm
        confirm = False
        dialog.destroy()

    button_no = ctk.CTkButton(
        buttons_frame,
        text="No",
        command=on_no,
        font=("Impact", 14),
        fg_color="red",
        width=100
    )
    button_no.pack(side="right", padx=20, pady=10)

    # Wait for the dialog to close
    dialog.wait_window()

    return confirm



# Back Button (Placed at the bottom left corner)
button_back_delete_ci = ctk.CTkButton(
    delete_ci_frame,
    text="Back",
    command=go_to_dashboard,  # Function to go back to the dashboard
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_delete_ci.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner


# Update CI Frame
update_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
update_ci_frame.pack_forget()

# Update CI Title (Maximized size and moved to the left top corner)
label_title_update_ci = ctk.CTkLabel(update_ci_frame, text="CONFIGURATION ITEM - UPDATE", font=("Impact", 60))
label_title_update_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the Update CI title)
horizontal_line_update_ci = ctk.CTkFrame(update_ci_frame, height=6, fg_color="#00008B")
horizontal_line_update_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Blue Toolbar Box
toolbar_box_update_ci = ctk.CTkFrame(
    update_ci_frame,
    border_width=1,  # Border thickness
    border_color="#00008B",  # Border color
    fg_color="#00008B",  # Background color of the box
    height=60,
    width=1200  # Set the height of the toolbar box (adjust as needed)
)
toolbar_box_update_ci.pack(pady=110, fill="both")  # Padding above and below the box, and stretch horizontally

# Buttons for the Toolbar (Object type group, Dashboard, Dependencies, Relationship, Charts)
toolbar_buttons_update_ci = ["Object type group", "Dashboard", "Dependencies", "Relationship", "Charts"]

# Place buttons on the toolbar
for i, button_text in enumerate(toolbar_buttons_update_ci):
    button = ctk.CTkButton(
        toolbar_box_update_ci,
        text=button_text,
        font=("Impact", 14),
        fg_color="transparent",  # Transparent background
        hover_color="#1f538d",  # Hover color
        width=120,  # Button width
        height=30,  # Button height
        corner_radius=10
    )
    if button_text == "Object type group":
        button.configure(command=lambda: show_object_type_buttons_update_ci())  # Bind the "Object type group" button to the function
    button.grid(row=0, column=i, padx=90, pady=5)  # Place buttons in a grid layout


# Frame to hold CI type buttons in Update CI page
ci_type_buttons_frame_update_ci = ctk.CTkFrame(update_ci_frame, fg_color="transparent")
ci_type_buttons_frame_update_ci.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame
ci_type_buttons_frame_update_ci.pack_forget()  # Initially hidden

# Frame to hold Object type group buttons
object_type_buttons_frame_update_ci = ctk.CTkFrame(update_ci_frame, fg_color="transparent")
object_type_buttons_frame_update_ci.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame
object_type_buttons_frame_update_ci.pack_forget()  # Initially hidden


# Function to show Object type group buttons in Update CI page
def show_object_type_buttons_update_ci():
    print("Show Object Type Buttons Update CI function called")  # Debugging

    # Hide the CI type buttons frame
    ci_type_buttons_frame_update_ci.pack_forget()

    # Clear any existing buttons in the Object type group buttons frame
    for widget in object_type_buttons_frame_update_ci.winfo_children():
        widget.destroy()

    # List of Object type group buttons
    object_type_buttons = ["Software", "Hardware", "Network", "Documentation", "Information", "Cloud", "Facilities"]

    # Create and display the Object type group buttons
    for i, button_text in enumerate(object_type_buttons):
        if button_text == "Software":
            # Create the "Software" button with the new command
            button = ctk.CTkButton(
                object_type_buttons_frame_update_ci,
                text=button_text,
                font=("Impact", 16),
                fg_color="#1f538d",
                hover_color="#00008B",
                width=150,
                height=40,
                corner_radius=10,
                command=go_to_software_ci_update  # Bind the "Software" button to the new function
            )
        else:
            # Create other buttons with no command or a default command
            button = ctk.CTkButton(
                object_type_buttons_frame_update_ci,
                text=button_text,
                font=("Impact", 16),
                fg_color="#1f538d",
                hover_color="#00008B",
                width=150,
                height=40,
                corner_radius=10
            )
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)  # Arrange buttons in a grid (3 columns)

    # Show the Object type group buttons frame
    object_type_buttons_frame_update_ci.pack(expand=True, padx=300, pady=(0, 50), fill="both")
    print("Object Type Buttons displayed")  # Debugging

# Function to show CI type buttons in Update CI page
def show_ci_type_buttons_update_ci():
    print("Show CI Type Buttons Update CI function called")  # Debugging

    # Hide the Object type group buttons frame
    object_type_buttons_frame_update_ci.pack_forget()

    # Clear any existing buttons in the CI type buttons frame
    for widget in ci_type_buttons_frame_update_ci.winfo_children():
        widget.destroy()

    # List of CI type buttons
    ci_type_buttons = ["Application", "Database", "OS", "API's", "Virtual Machine", "Software Licenses"]

    # Create and display the CI type buttons
    for i, button_text in enumerate(ci_type_buttons):
        button = ctk.CTkButton(
            ci_type_buttons_frame_update_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10  # Rounded corners
        )
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)  # Arrange buttons in a grid

    # Ensure the frame is visible
    ci_type_buttons_frame_update_ci.pack(expand=True, padx=300, pady=(0, 50), fill="both")
    print("CI Type Buttons displayed")  # Debugging

# Back Button (Placed at the bottom left corner)
button_back_update_ci = ctk.CTkButton(
    update_ci_frame,
    text="Back",
    command=go_to_dashboard,  # Directly call the go_to_dashboard function
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_update_ci.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner


# View CI Frame
view_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
view_ci_frame.pack_forget()


# View CI Title (Maximized size and moved to the left top corner)
label_title_view_ci = ctk.CTkLabel(view_ci_frame, text="CONFIGURATION ITEM-VIEW", font=("Impact", 60))
label_title_view_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the View CI title)
horizontal_line_view_ci = ctk.CTkFrame(view_ci_frame, height=6, fg_color="#00008B")
horizontal_line_view_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Blue Toolbar Box
toolbar_box_view_ci = ctk.CTkFrame(
    view_ci_frame,
    border_width=1,  # Border thickness
    border_color="#00008B",  # Border color
    fg_color="#00008B",  # Background color of the box
    height=60,
    width=1200  # Set the height of the toolbar box (adjust as needed)
)
toolbar_box_view_ci.pack(pady=110, fill="both")  # Padding above and below the box, and stretch horizontally



# Buttons for the Toolbar (Object type group, Dashboard, Dependencies, Relationship, Charts)
toolbar_buttons_view_ci = ["Object type group", "Dashboard", "Dependencies", "Relationship", "Charts"]

# Place buttons on the toolbar
for i, button_text in enumerate(toolbar_buttons_view_ci):
    button = ctk.CTkButton(
        toolbar_box_view_ci,
        text=button_text,
        font=("Impact", 14),
        fg_color="transparent",  # Transparent background
        hover_color="#1f538d",  # Hover color
        width=120,  # Button width
        height=30,  # Button height
        corner_radius=10
    )
    if button_text == "Object type group":
        button.configure(command=lambda: show_object_type_buttons_view_ci())  # Bind the "Object type group" button to the function
    button.grid(row=0, column=i, padx=90, pady=5)  # Place buttons in a grid layout






# Frame to hold CI type buttons in View CI page
ci_type_buttons_frame_view_ci = ctk.CTkFrame(view_ci_frame, fg_color="transparent")
ci_type_buttons_frame_view_ci.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame
ci_type_buttons_frame_view_ci.pack_forget()  # Initially hidden'''




# Frame to hold Object type group buttons
object_type_buttons_frame_view_ci = ctk.CTkFrame(view_ci_frame, fg_color="transparent")
object_type_buttons_frame_view_ci.place(relx=0.5, rely=0.5, anchor="center")  # Centered in the frame
object_type_buttons_frame_view_ci.pack_forget()  # Initially hidden



def show_object_type_buttons_view_ci():
    print("Show Object Type Buttons View CI function called")  # Debugging

    # Hide the CI type buttons frame
    ci_type_buttons_frame_view_ci.pack_forget()

    # Clear any existing buttons in the Object type group buttons frame
    for widget in object_type_buttons_frame_view_ci.winfo_children():
        widget.destroy()

    # List of Object type group buttons
    object_type_buttons = ["Software", "Hardware", "Network", "Documentation", "Information", "Cloud", "Facilities"]

    # Create and display the Object type group buttons
    for i, button_text in enumerate(object_type_buttons):
        if button_text == "Software":
            # Create the "Software" button with the new command
            button = ctk.CTkButton(
                object_type_buttons_frame_view_ci,
                text=button_text,
                font=("Impact", 16),
                fg_color="#1f538d",
                hover_color="#00008B",
                width=150,
                height=40,
                corner_radius=10,
                command=go_to_software_ci  # Bind the "Software" button to the new function
            )
        else:
            # Create other buttons with no command or a default command
            button = ctk.CTkButton(
                object_type_buttons_frame_view_ci,
                text=button_text,
                font=("Impact", 16),
                fg_color="#1f538d",
                hover_color="#00008B",
                width=150,
                height=40,
                corner_radius=10
            )
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)  # Arrange buttons in a grid (3 columns)

    # Show the Object type group buttons frame
    object_type_buttons_frame_view_ci.pack(expand=True, padx=300, pady=(0, 50), fill="both")
    print("Object Type Buttons displayed")  # Debugging

# Function to show CI type buttons in View CI page
def show_ci_type_buttons_view_ci():
    print("Show CI Type Buttons View CI function called")  # Debugging

    # Hide the Object type group buttons frame
    object_type_buttons_frame_view_ci.pack_forget()

    # Clear any existing buttons in the CI type buttons frame
    for widget in ci_type_buttons_frame_view_ci.winfo_children():
        widget.destroy()

    # List of CI type buttons
    ci_type_buttons = ["Application", "Database", "OS", "API's", "Virtual Machine", "Software Licenses"]

    # Create and display the CI type buttons
    for i, button_text in enumerate(ci_type_buttons):
        button = ctk.CTkButton(
            ci_type_buttons_frame_view_ci,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10  # Rounded corners
        )
        button.grid(row=i // 3, column=i % 3, padx=10, pady=10)  # Arrange buttons in a grid

    # Ensure the frame is visible
    ci_type_buttons_frame_view_ci.pack(expand=True, padx=300, pady=(0, 50), fill="both")
    print("CI Type Buttons displayed")  # Debugging

# Back Button (Placed at the bottom left corner)
button_back_view_ci = ctk.CTkButton(
    view_ci_frame,
    text="Back",
    command=go_to_dashboard,  # Directly call the go_to_dashboard function
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_view_ci.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner


# Database CI Title (Maximized size and moved to the left top corner)
label_title_database_ci = ctk.CTkLabel(database_ci_frame, text="DATABASE CI - ADD", font=("Impact", 60))
label_title_database_ci.place(relx=0.05, rely=0.05, anchor="nw")  # Positioned at the left top corner

# Horizontal Line (Below the Database CI title)
horizontal_line_database_ci = ctk.CTkFrame(database_ci_frame, height=6, fg_color="#00008B")
horizontal_line_database_ci.place(rely=0.20, relwidth=1.0, anchor="nw")  # Positioned below the title

# Entry Fields for Database CI (Left Side)
left_entry_fields_db = [
    ("Name", 0.05, 0.30),  # Adjusted rely from 0.25 to 0.30
    ("Type", 0.05, 0.40),  # Adjusted rely from 0.35 to 0.40
    ("Version", 0.05, 0.50),  # Adjusted rely from 0.45 to 0.50
    ("Environment", 0.05, 0.60),  # Adjusted rely from 0.55 to 0.60
    ("Status", 0.05, 0.70),  # Adjusted rely from 0.65 to 0.70
]

# Entry Fields for Database CI (Right Side)
right_entry_fields_db = [
    ("Schema", 0.55, 0.30),  # Adjusted rely from 0.25 to 0.30
    ("Owner", 0.55, 0.40),  # Adjusted rely from 0.35 to 0.40
    ("Backup details", 0.55, 0.50),  # Adjusted rely from 0.45 to 0.50
    ("Dependencies", 0.05, 0.80),  # New field for Dependencies
]

# Create entry fields (Left Side)
entries_db = {}
for field, relx, rely in left_entry_fields_db:
    label = ctk.CTkLabel(database_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(database_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_db[field.lower().replace(" ", "_")] = entry

# Create entry fields (Right Side)
for field, relx, rely in right_entry_fields_db:
    label = ctk.CTkLabel(database_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(database_ci_frame, width=250, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_db[field.lower().replace(" ", "_")] = entry


# Description Box (Right Side)
label_description_db = ctk.CTkLabel(database_ci_frame, text="Description", font=("Impact", 16))
label_description_db.place(relx=0.55, rely=0.80, anchor="w")

text_description_db = ctk.CTkTextbox(database_ci_frame, width=250, height=40, font=("Impact", 14))
text_description_db.place(relx=0.55 + 0.15, rely=0.80, anchor="w")

# Buttons (Save, Cancel, Add Custom Fields)
button_save_db = ctk.CTkButton(
    database_ci_frame,
    text="Save",
    command=lambda: save_database_ci(),
    width=120,
    height=40,
    font=("Impact", 18),
    fg_color="green"
)
button_save_db.place(relx=0.25, rely=0.95, anchor="sw")


# Function to clear Database CI fields
def clear_database_ci_fields():
    for entry in entries_db.values():
        entry.delete(0, "end")
    text_description_db.delete("1.0", "end")

button_cancel_db = ctk.CTkButton(
    database_ci_frame,
    text="Cancel",
    command=lambda: clear_database_ci_fields(),
    width=120,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_cancel_db.place(relx=0.45, rely=0.95, anchor="sw")

button_add_custom_fields_db = ctk.CTkButton(
    database_ci_frame,
    text="Add Custom Fields",
    command=lambda: show_info("Info", "Custom fields functionality not implemented yet."),
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="#1f538d"
)
button_add_custom_fields_db.place(relx=0.70, rely=0.95, anchor="sw")

# Back Button (Placed at the bottom left corner)
button_back_database_ci = ctk.CTkButton(
    database_ci_frame,
    text="Back",
    command=lambda: go_to_add_ci(),  # Function to go back to the Add CI frame
    width=120,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_database_ci.place(relx=0.05, rely=0.95, anchor="sw")  # Positioned at the bottom left corner



# Software CI Frame
software_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
software_ci_frame.pack_forget()




# Back Button (Placed next to the Software CI title)
button_back_software_ci = ctk.CTkButton(
    software_ci_frame,
    text="Back",
    command=lambda: go_to_view_ci(),  # Function to go back to the View CI frame
    width=150,  # Button width
    height=40,  # Button height
    font=("Impact", 18),  # Font size
    fg_color="red"  # Button color
)
button_back_software_ci.place(relx=0.80, rely=0.05, anchor="nw")  # Positioned next to the title






# Title for Software CI Page
label_title_software_ci = ctk.CTkLabel(software_ci_frame, text="SOFTWARE CI", font=("Impact", 60))
label_title_software_ci.place(relx=0.05, rely=0.05, anchor="nw")

# Horizontal Line (Below the Software CI title)
horizontal_line_software_ci = ctk.CTkFrame(software_ci_frame, height=6, fg_color="#00008B")
horizontal_line_software_ci.place(rely=0.20, relwidth=1.0, anchor="nw")

# Search Bar and Search Button
search_entry = ctk.CTkEntry(
    software_ci_frame,
    placeholder_text="Search CI...",
    width=400,
    height=40,
    font=("Impact", 14)
)
search_entry.place(relx=0.3, rely=0.25, anchor="nw")  # Positioned below the horizontal line

search_button = ctk.CTkButton(
    software_ci_frame,
    text="Search",
    font=("Impact", 14),
    fg_color="#1f538d",
    hover_color="#00008B",
    width=100,
    height=40,
    corner_radius=10,
    command=lambda: search_ci(search_entry.get())  # Bind the search function
)
search_button.place(relx=0.6, rely=0.25, anchor="nw")  # Positioned next to the search bar

clear_button = ctk.CTkButton(
    software_ci_frame,
    text="Clear",
    font=("Impact", 14),
    fg_color="red",
    hover_color="#8B0000",
    width=100,
    height=40,
    corner_radius=10,
    command=lambda: clear_search()  # Bind the clear function
)
clear_button.place(relx=0.7, rely=0.25, anchor="nw")  # Positioned next to the search button

scrollable_table_frame_software_ci = ctk.CTkScrollableFrame(
    software_ci_frame,
    fg_color="transparent",  # Add a border for debugging # Make the border visible
    width=760,  # Adjust width as needed
    height=280,  # Adjust height as needed
    orientation="horizontal"  # Ensure horizontal scrolling
)
scrollable_table_frame_software_ci.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar


def go_to_os_ci():
    # Hide the Add CI frame
    add_ci_frame.pack_forget()

    # Show the OS CI frame
    os_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

def go_to_add_ci():
    # Hide all other frames
    application_ci_frame.pack_forget()  # Hide the Application CI frame
    database_ci_frame.pack_forget()     # Hide the Database CI frame (if applicable)
    os_ci_frame.pack_forget()           # Hide the OS CI frame (if applicable)

    # Show the Add CI frame
    add_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

def go_to_api_ci():
    add_ci_frame.pack_forget()
    api_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

# OS CI Frame
os_ci_frame = ctk.CTkFrame(root, width=1200, height=800, border_width=5, border_color="#00008B")
os_ci_frame.pack_forget()

# OS CI Title
label_title_os_ci = ctk.CTkLabel(os_ci_frame, text="OPERATING SYSTEM CI - ADD", font=("Impact", 60))
label_title_os_ci.place(relx=0.05, rely=0.05, anchor="nw")

# Horizontal Line (Below the OS CI title)
horizontal_line_os_ci = ctk.CTkFrame(os_ci_frame, height=6, fg_color="#00008B")
horizontal_line_os_ci.place(rely=0.20, relwidth=1.0, anchor="nw")

# Entry Fields for OS CI (Left Side)
left_entry_fields_os = [
    ("Name", 0.05, 0.30),
    ("Version", 0.05, 0.40),
    ("Edition", 0.05, 0.50),
    ("Architecture", 0.05, 0.60),
    ("Kernel Version", 0.05, 0.70),
]

# Entry Fields for OS CI (Right Side)
right_entry_fields_os = [
    ("License", 0.55, 0.30),
    ("IP Address", 0.55, 0.40),
    ("Status", 0.55, 0.50),
    ("Owner", 0.55, 0.60),
    ("Patch Level", 0.55, 0.70),
    ("Installation Date", 0.55, 0.80),
]

# Create entry fields (Left Side)
entries_os = {}
for field, relx, rely in left_entry_fields_os:
    label = ctk.CTkLabel(os_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(os_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_os[field.lower().replace(" ", "_")] = entry

# Create entry fields (Right Side)
for field, relx, rely in right_entry_fields_os:
    label = ctk.CTkLabel(os_ci_frame, text=field, font=("Impact", 16))
    label.place(relx=relx, rely=rely, anchor="w")

    entry = ctk.CTkEntry(os_ci_frame, width=300, height=40, font=("Impact", 14))
    entry.place(relx=relx + 0.15, rely=rely, anchor="w")
    entries_os[field.lower().replace(" ", "_")] = entry

# Description Box (Right Side)
label_description_os = ctk.CTkLabel(os_ci_frame, text="Description", font=("Impact", 16))
label_description_os.place(relx=0.55, rely=0.80, anchor="w")

text_description_os = ctk.CTkTextbox(os_ci_frame, width=300, height=50, font=("Impact", 14))
text_description_os.place(relx=0.55 + 0.15, rely=0.80, anchor="w")

# Buttons (Save, Cancel, Back)
button_save_os = ctk.CTkButton(
    os_ci_frame,
    text="Save",
    command=lambda: save_os_ci(),
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="green"
)
button_save_os.place(relx=0.25, rely=0.95, anchor="sw")

def clear_os_ci_fields():
    # Clear all entry fields in the OS CI page
    for entry in entries_os.values():
        entry.delete(0, "end")  # Clear text in entry fields
    text_description_os.delete("1.0", "end")  # Clear the description textbox

button_cancel_os = ctk.CTkButton(
    os_ci_frame,
    text="Cancel",
    command=lambda: clear_os_ci_fields(),
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_cancel_os.place(relx=0.45, rely=0.95, anchor="sw")

button_back_os = ctk.CTkButton(
    os_ci_frame,
    text="Back",
    command=lambda: go_to_add_ci(),
    width=150,
    height=40,
    font=("Impact", 18),
    fg_color="red"
)
button_back_os.place(relx=0.05, rely=0.95, anchor="sw")

def display_application_table():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT app_name, environment, version, owner, status, dependencies, ci_id, app_license, description
        FROM application_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Application Data:", data)

        # Define column headers
        headers = ["App Name", "Environment", "Version", "Owner", "Status", "Dependencies", "CI ID", "App License", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame, table_frame
        scrollable_table_frame = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame.place(relx=0.3, rely=0.40, anchor="nw")

        # Create the table inside the scrollable frame
        table_frame = ctk.CTkFrame(scrollable_table_frame, fg_color="transparent")
        table_frame.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching application data:", e)

def display_database_table():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, type, version, environment, status, schema, owner, backup_details, dependencies, ci_id, description
        FROM database_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Database Data:", data)

        if not data:
            print("No data found in the database_cis table.")
            return

        # Define column headers
        headers = ["Name", "Type", "Version", "Environment", "Status", "Schema", "Owner", "Backup Details", "Dependencies", "CI ID", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_db, table_frame_db
        scrollable_table_frame_db = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_db.place(relx=0.3, rely=0.40, anchor="nw")

        # Create the table inside the scrollable frame
        table_frame_db = ctk.CTkFrame(scrollable_table_frame_db, fg_color="transparent")
        table_frame_db.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_db,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_db.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching database data:", e)

def clear_table_frame():
    # Destroy the existing table frame if it exists
    if 'scrollable_table_frame_vm' in globals():
        scrollable_table_frame_vm.destroy()
    if 'scrollable_table_frame' in globals():
        scrollable_table_frame.destroy()
    if 'scrollable_table_frame_db' in globals():
        scrollable_table_frame_db.destroy()
    if 'scrollable_table_frame_os' in globals():
        scrollable_table_frame_os.destroy()
    if 'scrollable_table_frame_api' in globals():
        scrollable_table_frame_api.destroy()

def display_os_table():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, version, edition, architecture, kernel_version, license, ip_address, status, owner, patch_level, installation_date, ci_id, description
        FROM os_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched OS Data:", data)

        # Define column headers
        headers = ["Name", "Version", "Edition", "Architecture", "Kernel Version", "License", "IP Address", "Status", "Owner", "Patch Level", "Installation Date", "CI ID", "Description"]

        # Combine headers and data
        table_data = [headers] + data

        # Debugging: Print table data
        print("Table Data:", table_data)

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_os, table_frame_os
        scrollable_table_frame_os = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_os.place(relx=0.3, rely=0.40, anchor="nw")

        # Debugging: Print scrollable frame placement
        print("Scrollable Frame placed at relx=0.3, rely=0.40")

        # Create the table inside the scrollable frame
        table_frame_os = ctk.CTkFrame(scrollable_table_frame_os, fg_color="transparent")
        table_frame_os.pack(fill="both", expand=True)

        # Debugging: Print table frame creation
        print("Table Frame created")

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Debugging: Print column widths
        print("Column Widths:", column_widths)

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_os,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Debugging: Print table cells created
        print("Table cells created")

        # Adjust the table frame to fit the content
        table_frame_os.update_idletasks()

        # Debugging: Print table frame updated
        print("Table frame updated")
    except sqlite3.Error as e:
        print("Error fetching OS data:", e)

def display_api_table():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, version, type, protocol, endpoint_url, status, vendor, dependencies, environment, authentication_method, description, ci_id
        FROM api_cis
        """)
        data = cursor.fetchall()

        # Define column headers
        headers = ["Name", "Version", "Type", "Protocol", "Endpoint URL", "Status", "Vendor", "Dependencies", "Environment", "Authentication Method", "Description", "CI ID"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_api, table_frame_api
        scrollable_table_frame_api = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_api.place(relx=0.3, rely=0.40, anchor="nw")

        # Create the table inside the scrollable frame
        table_frame_api = ctk.CTkFrame(scrollable_table_frame_api, fg_color="transparent")
        table_frame_api.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_api,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_api.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching API data:", e)

def display_vm_table():
    try:
        # Fetch data from the database
        cursor.execute("""
        SELECT name, host_machine, version, memory_size, status, vm_template, snapshots, disk, creation_date, dependencies, description, ci_id
        FROM vm_cis
        """)
        data = cursor.fetchall()

        # Debugging: Print the fetched data
        print("Fetched Virtual Machine Data:", data)

        # Define column headers
        headers = ["Name", "Host Machine", "Version", "Memory Size", "Status", "VM Template", "Snapshots", "Disk", "Creation Date", "Dependencies", "Description", "CI ID"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame_vm, table_frame_vm
        scrollable_table_frame_vm = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame_vm.place(relx=0.3, rely=0.40, anchor="nw")

        # Create the table inside the scrollable frame
        table_frame_vm = ctk.CTkFrame(scrollable_table_frame_vm, fg_color="transparent")
        table_frame_vm.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame_vm,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame_vm.update_idletasks()
    except sqlite3.Error as e:
        print("Error fetching Virtual Machine data:", e)

# Scrollable Frame to hold CI type buttons in Software CI page
ci_type_buttons_scrollable_frame = ctk.CTkScrollableFrame(
    software_ci_frame,
    fg_color="transparent",
    width=200,  # Adjust width as needed
    height=300  # Adjust height to fit within the frame
)
ci_type_buttons_scrollable_frame.place(relx=0.05, rely=0.25, anchor="nw")  # Positioned below the horizontal line

# List of CI type buttons
ci_type_buttons = ["Application", "Database", "OS", "API's", "Virtual Machine", "Software Licenses"]

# Create and display the CI type buttons
for i, button_text in enumerate(ci_type_buttons):
    if button_text == "Application":
        # Application button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=display_application_table  # Bind to display_application_table
        )
    elif button_text == "Database":
        # Database button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=display_database_table  # Bind to display_database_table
        )
    elif button_text == "OS":
        # OS button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=display_os_table  # Bind to display_os_table
        )
    elif button_text == "API's":
        # API's button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=display_api_table  # Bind to display_api_table
        )
    elif button_text == "Virtual Machine":
        # Virtual Machine button
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=display_vm_table  # Bind to display_vm_table
        )
    else:
        # Other buttons (e.g., Software Licenses)
        button = ctk.CTkButton(
            ci_type_buttons_scrollable_frame,
            text=button_text,
            font=("Impact", 14),
            fg_color="#1f538d",  # Button color
            hover_color="#00008B",  # Hover color
            width=150,  # Button width
            height=40,  # Button height
            corner_radius=10,  # Rounded corners
            command=lambda bt=button_text: show_info("Info", f"{bt} functionality not implemented yet.")
        )

    # Arrange buttons vertically (one below the other)
    button.grid(row=i, column=0, padx=10, pady=10)


# Vertical Line (Right after the scrollable frame)
vertical_line = ctk.CTkFrame(
    software_ci_frame,
    width=4,  # Line width
    height=600,  # Match the height of the scrollable frame
    fg_color="#00008B"  # Line color
)
vertical_line.place(relx=0.25, rely=0.20, anchor="nw")  # Positioned right after the scrollable frame


def search_ci(search_term):
    try:
        print(f"Searching for CI Name: {search_term}")  # Debugging: Print the search term

        # Ensure the database connection is open
        if not conn:
            print("Database connection is not open.")
            return

        # Query to search across all CI tables using only the CI Name
        query = """
        SELECT 'Application' AS ci_type, app_name AS name
        FROM application_cis
        WHERE app_name LIKE ?
        UNION ALL
        SELECT 'Database' AS ci_type, name
        FROM database_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'API' AS ci_type, name
        FROM api_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'OS' AS ci_type, name
        FROM os_cis
        WHERE name LIKE ?
        UNION ALL
        SELECT 'VM' AS ci_type, name
        FROM vm_cis
        WHERE name LIKE ?
        """

        # Execute the query with the search term (only searching by CI Name)
        cursor.execute(query, (
            f"%{search_term}%",  # Application
            f"%{search_term}%",  # Database
            f"%{search_term}%",  # API
            f"%{search_term}%",  # OS
            f"%{search_term}%"   # VM
        ))
        data = cursor.fetchall()

        print(f"Fetched Data: {data}")  # Debugging: Print the fetched data

        if not data:
            print("No matching data found.")  # Debugging: Print if no data is found
            show_info("Info", "No matching data found.")
            return

        # Clear the existing table
        clear_table_frame()

        # Define column headers
        headers = ["CI Type", "Name"]

        # Combine headers and data
        table_data = [headers] + data

        # Create a horizontal scrollable frame to hold the table
        global scrollable_table_frame, table_frame
        scrollable_table_frame = ctk.CTkScrollableFrame(
            software_ci_frame,
            fg_color="transparent",
            width=760,
            height=280,
            orientation="horizontal"
        )
        scrollable_table_frame.place(relx=0.3, rely=0.40, anchor="nw")  # Positioned below the search bar

        # Create the table inside the scrollable frame
        table_frame = ctk.CTkFrame(scrollable_table_frame, fg_color="transparent")
        table_frame.pack(fill="both", expand=True)

        # Calculate the maximum width required for each column
        column_widths = [max(len(str(row[col_idx])) for row in table_data) for col_idx in range(len(headers))]

        # Create labels for each cell in the table
        for row_idx, row in enumerate(table_data):
            for col_idx, cell_value in enumerate(row):
                # Create a label for each cell
                cell = ctk.CTkLabel(
                    table_frame,
                    text=str(cell_value),
                    font=("Impact", 12),
                    width=column_widths[col_idx] * 8,  # Adjust width dynamically based on content
                    height=30,
                    fg_color="#1f538d" if row_idx == 0 else "transparent",  # Header row color
                    text_color="white",
                    corner_radius=5 if row_idx == 0 else 0  # Rounded corners for header
                )
                cell.grid(row=row_idx, column=col_idx, padx=2, pady=2, sticky="w")  # Align text to the left

        # Adjust the table frame to fit the content
        table_frame.update_idletasks()
    except sqlite3.Error as e:
        print("Error searching CI data:", e)
        show_error("Error", f"Failed to search data: {e}")





def clear_table_frame():
    # Destroy the existing table frame if it exists
    if 'scrollable_table_frame_vm' in globals():
        scrollable_table_frame_vm.destroy()
    if 'scrollable_table_frame' in globals():
        scrollable_table_frame.destroy()
    if 'scrollable_table_frame_db' in globals():
        scrollable_table_frame_db.destroy()
    if 'scrollable_table_frame_os' in globals():
        scrollable_table_frame_os.destroy()
    if 'scrollable_table_frame_api' in globals():
        scrollable_table_frame_api.destroy()

def clear_search():
    # Clear the search entry
    search_entry.delete(0, "end")

    # Display the full table again
    display_application_table()



def go_to_software_ci():
    # Hide all other frames
    view_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    add_ci_frame.pack_forget()
    dashboard_frame.pack_forget()
    login_frame.pack_forget()
    home_frame.pack_forget()

    # Show the Software CI frame
    software_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

    # Display the Application table by default when the frame is loaded
    display_application_table()

def go_to_view_ci():
    # Hide all other frames
    software_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    add_ci_frame.pack_forget()
    dashboard_frame.pack_forget()
    login_frame.pack_forget()
    home_frame.pack_forget()


    # Show the View CI frame
    view_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

def go_to_software_ci_update():
    # Hide all other frames
    update_ci_frame.pack_forget()
    application_ci_frame.pack_forget()
    database_ci_frame.pack_forget()
    add_ci_frame.pack_forget()
    dashboard_frame.pack_forget()
    login_frame.pack_forget()
    home_frame.pack_forget()

    # Show the Software CI frame
    software_ci_frame.pack(expand=True, padx=50, pady=50, fill="both")

    # Display the Application table by default when the frame is loaded
    display_application_table()  # Ensure this function is defined and works correctly

# Main loop
root.mainloop()
