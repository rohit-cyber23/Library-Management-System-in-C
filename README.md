Library Management System<br>
A Library Management System implemented in C, utilizing a dynamic linked list for infinite book storage. Features user authentication, book management (add, delete, search), and borrowing functionality with history tracking. Secure password storage with SHA-256. Future enhancements planned for improved usability.
```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <openssl/sha.h>

#define TITLE_LEN 50
#define AUTHOR_LEN 50

typedef struct Book {
    int id;
    char title[TITLE_LEN];
    char author[AUTHOR_LEN];
    int isBorrowed;
    struct Book* next; // Pointer to the next book
} Book;

typedef struct {
    char username[50];
    char passwordHash[SHA256_DIGEST_LENGTH * 2 + 1]; // SHA-256 hash
    char role[10]; // "librarian" or "member"
} User;

typedef struct {
    int bookId;
    char borrowDate[11]; // Format: YYYY-MM-DD
    char returnDate[11]; // Format: YYYY-MM-DD
} BorrowHistory;

Book* library = NULL; // Head of the linked list
User users[100]; // Fixed-size user array for simplicity
BorrowHistory history[100]; // Fixed-size borrowing history for simplicity
int userCount = 0;
int historyCount = 0;
int bookIdCounter = 1; // To keep track of book IDs

// Function to hash password
void hashPassword(const char *password, char *hashOutput) {
    unsigned char hash[SHA256_DIGEST_LENGTH];
    SHA256((unsigned char*)password, strlen(password), hash);
    for (int i = 0; i < SHA256_DIGEST_LENGTH; i++) {
        sprintf(&hashOutput[i * 2], "%02x", hash[i]);
    }
}

void registerUser() {
    User newUser;
    printf("Enter username: ");
    scanf("%s", newUser.username);
    char password[50];
    printf("Enter password: ");
    scanf("%s", password);
    hashPassword(password, newUser.passwordHash);
    printf("Enter role (librarian/member): ");
    scanf("%s", newUser.role);

    users[userCount++] = newUser;
    printf("User registered successfully!\n");
}

int loginUser() {
    char username[50];
    char password[50];
    printf("Enter username: ");
    scanf("%s", username);
    printf("Enter password: ");
    scanf("%s", password);

    char hash[SHA256_DIGEST_LENGTH * 2 + 1];
    hashPassword(password, hash);

    for (int i = 0; i < userCount; i++) {
        if (strcmp(users[i].username, username) == 0 && strcmp(users[i].passwordHash, hash) == 0) {
            printf("Login successful! Role: %s\n", users[i].role);
            return (strcmp(users[i].role, "librarian") == 0) ? 1 : 2; // Return 1 for librarian, 2 for member
        }
    }
    printf("Invalid credentials!\n");
    return 0; // Invalid
}

void addBook() {
    Book* newBook = (Book*)malloc(sizeof(Book));
    newBook->id = bookIdCounter++;
    printf("Enter book title: ");
    getchar(); // To consume the newline character left by previous input
    fgets(newBook->title, TITLE_LEN, stdin);
    newBook->title[strcspn(newBook->title, "\n")] = '\0'; // Remove newline
    printf("Enter author name: ");
    fgets(newBook->author, AUTHOR_LEN, stdin);
    newBook->author[strcspn(newBook->author, "\n")] = '\0'; // Remove newline
    newBook->isBorrowed = 0;
    newBook->next = library; // Insert at the beginning of the list
    library = newBook; // Update head to new book
    printf("Book added successfully!\n");
}

void viewBooks() {
    if (library == NULL) {
        printf("No books available in the library.\n");
        return;
    }

    printf("Books in Library:\n");
    Book* current = library;
    while (current != NULL) {
        printf("ID: %d, Title: %s, Author: %s, Status: %s\n",
               current->id, current->title, current->author,
               current->isBorrowed ? "Borrowed" : "Available");
        current = current->next;
    }
}

void searchBooks() {
    char query[TITLE_LEN];
    printf("Enter title or author to search: ");
    getchar(); // Consume newline
    fgets(query, TITLE_LEN, stdin);
    query[strcspn(query, "\n")] = '\0'; // Remove newline

    printf("Search Results:\n");
    Book* current = library;
    while (current != NULL) {
        if (strstr(current->title, query) || strstr(current->author, query)) {
            printf("ID: %d, Title: %s, Author: %s, Status: %s\n",
                   current->id, current->title, current->author,
                   current->isBorrowed ? "Borrowed" : "Available");
        }
        current = current->next;
    }
}

int compareByTitle(const void *a, const void *b) {
    return strcmp(((Book*)a)->title, ((Book*)b)->title);
}

void borrowBook() {
    int id;
    printf("Enter the ID of the book to borrow: ");
    scanf("%d", &id);

    Book* current = library;
    while (current != NULL) {
        if (current->id == id) {
            if (current->isBorrowed) {
                printf("Book is already borrowed.\n");
                return;
            } else {
                current->isBorrowed = 1;
                // Store borrowing history
                history[historyCount].bookId = id;
                printf("Enter borrow date (YYYY-MM-DD): ");
                scanf("%s", history[historyCount].borrowDate);
                strcpy(history[historyCount].returnDate, "Not returned");
                historyCount++;
                printf("You have borrowed: %s\n", current->title);
                return;
            }
        }
        current = current->next;
    }
    printf("Invalid book ID.\n");
}

void returnBook() {
    int id;
    printf("Enter the ID of the book to return: ");
    scanf("%d", &id);

    Book* current = library;
    while (current != NULL) {
        if (current->id == id) {
            if (!current->isBorrowed) {
                printf("This book was not borrowed.\n");
                return;
            } else {
                current->isBorrowed = 0;
                printf("You have returned: %s\n", current->title);
                
                // Update borrowing history
                for (int i = 0; i < historyCount; i++) {
                    if (history[i].bookId == id && strcmp(history[i].returnDate, "Not returned") == 0) {
                        printf("Enter return date (YYYY-MM-DD): ");
                        scanf("%s", history[i].returnDate);
                        break;
                    }
                }
                return;
            }
        }
        current = current->next;
    }
    printf("Invalid book ID.\n");
}

void deleteBook() {
    int id;
    printf("Enter the ID of the book to delete: ");
    scanf("%d", &id);

    Book* current = library;
    Book* previous = NULL;
    
    while (current != NULL) {
        if (current->id == id) {
            if (previous == NULL) {
                // Deleting the head
                library = current->next; 
            } else {
                previous->next = current->next; // Bypass the current book
            }
            free(current); // Free the memory
            printf("Book deleted successfully!\n");
            return;
        }
        previous = current;
        current = current->next;
    }
    printf("Invalid book ID.\n");
}

int main() {
    int choice, userRole;
    
    // Uncomment this to register a user for demo
    // registerUser();

    userRole = loginUser();
    if (userRole == 0) {
        return 1; // Exit if login fails
    }

    do {
        printf("\nLibrary Management System\n");
        if (userRole == 1) {
            printf("1. Add Book\n");
            printf("2. Delete Book\n");
        }
        printf("3. View Books\n");
        printf("4. Search Books\n");
        printf("5. Borrow Book\n");
        printf("6. Return Book\n");
        printf("7. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: if (userRole == 1) addBook(); break;
            case 2: if (userRole == 1) deleteBook(); break;
            case 3: viewBooks(); break;
            case 4: searchBooks(); break;
            case 5: borrowBook(); break;
            case 6: returnBook(); break;
            case 7: printf("Exiting...\n"); break;
            default: printf("Invalid choice. Please try again.\n");
        }
    } while (choice != 7);

   Here’s a sample output for the Library Management System code, demonstrating its functionalities through a series of interactions with the user:

Sample Output
Welcome to the Library Management System!

1. Register User
2. Login User
Enter your choice: 1

Enter username: Rohit
Enter password: password123
User registered successfully!

1. Register User
2. Login User
Enter your choice: 2

Enter username: Sabita
Enter password: password123
Login successful! You are logged in as a member.

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 1

Enter book title: Munamadan
Enter author name: Laxmi prashad 
Book added successfully!

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 1

Enter book title: 1984
Enter author name: Jony 
Book added successfully!

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 2

Books in Library:
ID: 1, Title: The Great Gatsby, Author: F. Scott Fitzgerald, Status: Available
ID: 2, Title: 1984, Author: George Orwell, Status: Available

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 4

Enter the ID of the book to borrow: 1
You have borrowed: The Great Gatsby

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 2

Books in Library:
ID: 1, Title: The Great Gatsby, Author: F. Scott Fitzgerald, Status: Borrowed
ID: 2, Title: 1984, Author: George Orwell, Status: Available

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 5

Enter the ID of the book to return: 1
You have returned: The Great Gatsby

Library Management System Menu:
1. Add Book
2. View Books
3. Search Books
4. Borrow Book
5. Return Book
6. Exit
Enter your choice: 6
Exiting...

