# 📚 ระบบบริหารจัดการห้องสมุด (Library Management System) - Python

## โครงสร้างโปรเจค
```
LibraryManagement/
├── models/
│   └── book.py
├── algorithms/
│   ├── sorting.py
│   └── searching.py
├── main.py
└── README.md
```

## 📄 book.py (Model Class)
```python
class Book:
    def __init__(self, isbn, title, author, publish_year):
        self.isbn = isbn
        self.title = title
        self.author = author
        self.publish_year = publish_year
    
    def __str__(self):
        return f"ISBN: {self.isbn} | ชื่อ: {self.title} | ผู้แต่ง: {self.author} | ปี: {self.publish_year}"
    
    def to_dict(self):
        return {
            'isbn': self.isbn,
            'title': self.title,
            'author': self.author,
            'publish_year': self.publish_year
        }
```

## 📄 sorting.py (Algorithms)
```python
import time

class SortingAlgorithms:
    
    @staticmethod
    def insertion_sort(books, key='publish_year'):
        """
        Insertion Sort - เหมาะสำหรับข้อมูลจำนวนน้อย
        Big O: O(n²) ใน worst case, O(n) ใน best case
        """
        n = len(books)
        for i in range(1, n):
            key_book = books[i]
            j = i - 1
            
            # เลื่อนข้อมูลที่มีค่าน้อยกว่าไปทางขวา
            while j >= 0 and getattr(books[j], key) > getattr(key_book, key):
                books[j + 1] = books[j]
                j -= 1
            books[j + 1] = key_book
        
        return books
    
    @staticmethod
    def merge_sort(books, key='publish_year'):
        """
        Merge Sort - เหมาะสำหรับข้อมูลจำนวนมาก
        Big O: O(n log n) ทุกกรณี
        """
        if len(books) <= 1:
            return books
        
        # แบ่งข้อมูลออกเป็นสองส่วน
        mid = len(books) // 2
        left = books[:mid]
        right = books[mid:]
        
        # เรียกใช้ recursive
        left = SortingAlgorithms.merge_sort(left, key)
        right = SortingAlgorithms.merge_sort(right, key)
        
        # รวมข้อมูลที่เรียงแล้ว
        return SortingAlgorithms._merge(left, right, key)
    
    @staticmethod
    def _merge(left, right, key):
        result = []
        i = j = 0
        
        while i < len(left) and j < len(right):
            if getattr(left[i], key) <= getattr(right[j], key):
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        
        # เพิ่มข้อมูลที่เหลือ
        result.extend(left[i:])
        result.extend(right[j:])
        
        return result
    
    @staticmethod
    def measure_time(sort_func, books, key):
        """วัดเวลาในการเรียงข้อมูล"""
        start_time = time.time()
        sorted_books = sort_func(books.copy(), key)
        end_time = time.time()
        return sorted_books, end_time - start_time
```

## 📄 searching.py (Search Algorithms)
```python
class SearchAlgorithms:
    
    @staticmethod
    def binary_search(books, target_isbn):
        """
        Binary Search - ค้นหาจาก ISBN (ข้อมูลต้องเรียงลำดับก่อน)
        Big O: O(log n)
        """
        left, right = 0, len(books) - 1
        
        while left <= right:
            mid = (left + right) // 2
            current_isbn = books[mid].isbn
            
            if current_isbn == target_isbn:
                return mid, books[mid]  # พบหนังสือ
            elif current_isbn < target_isbn:
                left = mid + 1
            else:
                right = mid - 1
        
        return -1, None  # ไม่พบหนังสือ
    
    @staticmethod
    def sequential_search(books, title_keyword):
        """
        Sequential Search - ค้นหาจากชื่อหนังสือ
        Big O: O(n)
        """
        found_books = []
        comparisons = 0
        
        for book in books:
            comparisons += 1
            if title_keyword.lower() in book.title.lower():
                found_books.append(book)
        
        return found_books, comparisons
```

## 📄 main.py (Main Program)
```python
from models.book import Book
from algorithms.sorting import SortingAlgorithms
from algorithms.searching import SearchAlgorithms
import json
import os

class LibraryManagementSystem:
    def __init__(self):
        self.books = []
        self.data_file = "library_data.json"
        self.load_data()
    
    def load_data(self):
        """โหลดข้อมูลจากไฟล์"""
        if os.path.exists(self.data_file):
            try:
                with open(self.data_file, 'r', encoding='utf-8') as f:
                    data = json.load(f)
                    self.books = [Book(**book_data) for book_data in data]
                print(f"✅ โหลดข้อมูลสำเร็จ: {len(self.books)} รายการ")
            except:
                self.books = []
                print("⚠️ ไม่พบข้อมูลเก่า เริ่มต้นด้วยข้อมูลว่าง")
    
    def save_data(self):
        """บันทึกข้อมูลลงไฟล์"""
        with open(self.data_file, 'w', encoding='utf-8') as f:
            data = [book.to_dict() for book in self.books]
            json.dump(data, f, ensure_ascii=False, indent=2)
        print("✅ บันทึกข้อมูลสำเร็จ")
    
    def add_book(self):
        """เพิ่มหนังสือใหม่"""
        print("\n📖 เพิ่มหนังสือใหม่")
        isbn = input("ISBN: ").strip()
        
        # ตรวจสอบ ISBN ซ้ำ
        for book in self.books:
            if book.isbn == isbn:
                print("❌ ISBN นี้มีอยู่ในระบบแล้ว")
                return
        
        title = input("ชื่อหนังสือ: ").strip()
        author = input("ผู้แต่ง: ").strip()
        
        try:
            publish_year = int(input("ปีที่พิมพ์: ").strip())
        except ValueError:
            print("❌ ปีที่พิมพ์ไม่ถูกต้อง")
            return
        
        new_book = Book(isbn, title, author, publish_year)
        self.books.append(new_book)
        self.save_data()
        print(f"✅ เพิ่มหนังสือ '{title}' สำเร็จ")
    
    def update_book(self):
        """แก้ไขข้อมูลหนังสือ"""
        print("\n✏️ แก้ไขข้อมูลหนังสือ")
        isbn = input("กรุณากรอก ISBN ของหนังสือที่ต้องการแก้ไข: ").strip()
        
        for book in self.books:
            if book.isbn == isbn:
                print(f"\nข้อมูลปัจจุบัน: {book}")
                print("\nกรุณากรอกข้อมูลใหม่ (ถ้าไม่ต้องการแก้ไข ให้กด Enter)")
                
                new_title = input(f"ชื่อหนังสือ [{book.title}]: ").strip()
                if new_title:
                    book.title = new_title
                
                new_author = input(f"ผู้แต่ง [{book.author}]: ").strip()
                if new_author:
                    book.author = new_author
                
                new_year = input(f"ปีที่พิมพ์ [{book.publish_year}]: ").strip()
                if new_year:
                    try:
                        book.publish_year = int(new_year)
                    except ValueError:
                        print("⚠️ ปีที่พิมพ์ไม่ถูกต้อง คงค่าเดิมไว้")
                
                self.save_data()
                print("✅ แก้ไขข้อมูลสำเร็จ")
                return
        
        print("❌ ไม่พบหนังสือที่มี ISBN นี้")
    
    def delete_book(self):
        """ลบหนังสือ"""
        print("\n🗑️ ลบหนังสือ")
        isbn = input("กรุณากรอก ISBN ของหนังสือที่ต้องการลบ: ").strip()
        
        for i, book in enumerate(self.books):
            if book.isbn == isbn:
                print(f"หนังสือที่จะลบ: {book}")
                confirm = input("ยืนยันการลบ? (y/n): ").strip().lower()
                
                if confirm == 'y':
                    deleted_book = self.books.pop(i)
                    self.save_data()
                    print(f"✅ ลบหนังสือ '{deleted_book.title}' สำเร็จ")
                else:
                    print("❌ ยกเลิกการลบ")
                return
        
        print("❌ ไม่พบหนังสือที่มี ISBN นี้")
    
    def display_books(self, books=None):
        """แสดงรายการหนังสือ"""
        if books is None:
            books = self.books
        
        if not books:
            print("\n📭 ไม่มีหนังสือในระบบ")
            return
        
        print(f"\n📚 รายการหนังสือทั้งหมด ({len(books)} รายการ)")
        print("-" * 80)
        for i, book in enumerate(books, 1):
            print(f"{i}. {book}")
        print("-" * 80)
    
    def sort_books_menu(self):
        """เมนูเรียงลำดับหนังสือ"""
        if not self.books:
            print("\n📭 ไม่มีหนังสือในระบบ")
            return
        
        print("\n🔀 เรียงลำดับหนังสือ")
        print("1. เรียงตามปีที่พิมพ์")
        print("2. เรียงตามชื่อผู้แต่ง")
        print("3. กลับไปเมนูหลัก")
        
        choice = input("เลือกตัวเลือก (1-3): ").strip()
        
        if choice == '1':
            key = 'publish_year'
            key_name = "ปีที่พิมพ์"
        elif choice == '2':
            key = 'author'
            key_name = "ชื่อผู้แต่ง"
        else:
            return
        
        # ทดสอบประสิทธิภาพของทั้งสองวิธี
        print(f"\n⏱️  ทดสอบประสิทธิภาพการเรียงลำดับตาม{key_name}")
        
        # Insertion Sort (สำหรับข้อมูลน้อย)
        if len(self.books) <= 50:
            sorted_books, time_taken = SortingAlgorithms.measure_time(
                SortingAlgorithms.insertion_sort, self.books, key
            )
            print(f"📊 Insertion Sort ใช้เวลา: {time_taken:.6f} วินาที")
            self.display_books(sorted_books)
        else:
            # Merge Sort (สำหรับข้อมูลมาก)
            sorted_books, time_taken = SortingAlgorithms.measure_time(
                SortingAlgorithms.merge_sort, self.books, key
            )
            print(f"📊 Merge Sort ใช้เวลา: {time_taken:.6f} วินาที")
            self.display_books(sorted_books)
    
    def search_books_menu(self):
        """เมนูค้นหาหนังสือ"""
        if not self.books:
            print("\n📭 ไม่มีหนังสือในระบบ")
            return
        
        print("\n🔍 ค้นหาหนังสือ")
        print("1. ค้นหาจาก ISBN (Binary Search)")
        print("2. ค้นหาจากชื่อหนังสือ (Sequential Search)")
        print("3. กลับไปเมนูหลัก")
        
        choice = input("เลือกตัวเลือก (1-3): ").strip()
        
        if choice == '1':
            # Binary Search (ต้องเรียง ISBN ก่อน)
            isbn = input("กรุณากรอก ISBN: ").strip()
            
            # เรียงหนังสือตาม ISBN ก่อนทำ Binary Search
            sorted_books = SortingAlgorithms.merge_sort(self.books, 'isbn')
            index, found_book = SearchAlgorithms.binary_search(sorted_books, isbn)
            
            if found_book:
                print(f"\n✅ พบหนังสือ: {found_book}")
            else:
                print(f"\n❌ ไม่พบหนังสือที่มี ISBN: {isbn}")
        
        elif choice == '2':
            # Sequential Search
            keyword = input("กรุณากรอกคำค้นหาจากชื่อหนังสือ: ").strip()
            found_books, comparisons = SearchAlgorithms.sequential_search(self.books, keyword)
            
            print(f"\n📊 จำนวนการเปรียบเทียบ: {comparisons} ครั้ง")
            if found_books:
                print(f"\n✅ พบ {len(found_books)} รายการ:")
                self.display_books(found_books)
            else:
                print(f"\n❌ ไม่พบหนังสือที่มีคำค้นหา: {keyword}")
    
    def run(self):
        """รันโปรแกรมหลัก"""
        while True:
            print("\n" + "="*50)
            print("📚 ระบบบริหารจัดการห้องสมุด")
            print("="*50)
            print(f"จำนวนหนังสือในระบบ: {len(self.books)} เล่ม")
            print("\n1. เพิ่มหนังสือ")
            print("2. แก้ไขข้อมูลหนังสือ")
            print("3. ลบหนังสือ")
            print("4. แสดงรายการหนังสือทั้งหมด")
            print("5. เรียงลำดับหนังสือ")
            print("6. ค้นหาหนังสือ")
            print("7. ออกจากโปรแกรม")
            
            choice = input("\nเลือกเมนู (1-7): ").strip()
            
            if choice == '1':
                self.add_book()
            elif choice == '2':
                self.update_book()
            elif choice == '3':
                self.delete_book()
            elif choice == '4':
                self.display_books()
            elif choice == '5':
                self.sort_books_menu()
            elif choice == '6':
                self.search_books_menu()
            elif choice == '7':
                self.save_data()
                print("\n👋 ขอบคุณที่ใช้บริการระบบบริหารจัดการห้องสมุด")
                break
            else:
                print("\n❌ กรุณาเลือกเมนู 1-7 เท่านั้น")

if __name__ == "__main__":
    app = LibraryManagementSystem()
    app.run()
```

## 📊 สรุป Big O ของแต่ละ Algorithm

### 1. **Insertion Sort** - O(n²)
- **Best Case**: O(n) - เมื่อข้อมูลเรียงอยู่แล้ว
- **Average Case**: O(n²)
- **Worst Case**: O(n²) - เมื่อข้อมูลเรียงย้อนกลับ
- **ใช้เมื่อ**: ข้อมูลจำนวนน้อย (≤ 50 รายการ)

### 2. **Merge Sort** - O(n log n)
- **ทุกกรณี**: O(n log n)
- **ใช้เมื่อ**: ข้อมูลจำนวนมาก (> 50 รายการ)
- **ข้อดี**: มีประสิทธิภาพคงที่ ไม่ขึ้นกับลักษณะข้อมูล

### 3. **Binary Search** - O(log n)
- **ต้องการ**: ข้อมูลต้องเรียงลำดับก่อนค้นหา
- **มีประสิทธิภาพสูง** เมื่อค้นหาด้วย ISBN

### 4. **Sequential Search** - O(n)
- **ไม่ต้องการ**: ข้อมูลเรียงลำดับ
- **ใช้เมื่อ**: ค้นหาจากชื่อหนังสือ (อาจมีหลายรายการ)

## 🚀 วิธีติดตั้งและรันโปรแกรม

1. **สร้างโฟลเดอร์โปรเจค:**
```bash
mkdir LibraryManagement
cd LibraryManagement
```

2. **สร้างโฟลเดอร์ย่อย:**
```bash
mkdir models algorithms
```

3. **สร้างไฟล์ตามโครงสร้างข้างต้น**

4. **รันโปรแกรม:**
```bash
python main.py
```

โปรแกรมนี้จะ:
- ✅ เก็บข้อมูลในไฟล์ JSON เพื่อให้ข้อมูลคงอยู่
- ✅ รองรับการเพิ่ม/แก้ไข/ลบ หนังสือ
- ✅ มีทั้ง Insertion Sort และ Merge Sort ให้เลือกตามขนาดข้อมูล
- ✅ ค้นหาได้ทั้งแบบ Binary Search (ISBN) และ Sequential Search (ชื่อ)
- ✅ แสดงเวลาในการทำงานของแต่ละ Algorithm
