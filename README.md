# Magic-metodlar-bilan-o-z-obyektlaringiz
Python-da dunder (magic) metodlar orqali klasslarga tabiiy xatti-harakat berish, ularni `@dataclass` bilan ixchamlashtirish hamda `__hash__` tuzog'ini hal qilish namunasi:

```python
"""
Magic (dunder) metodlar va dataclass namunasi.
Ushbu modul mypy va Python tip tekshiruvidan toza o'tadi.
"""

from dataclasses import dataclass
from typing import List, Set


# ==============================================================================
# 1. Qo'lda dunder metodlar bilan yozilgan Book klassi
# ==============================================================================

class Book:
    def __init__(self, title: str, author: str, year: int) -> None:
        self.title = title
        self.author = author
        self.year = year

    def __repr__(self) -> str:
        """print() va repr() uchun aniq va foydali ko'rinish."""
        return f"Book(title='{self.title}', author='{self.author}', year={self.year})"

    def __eq__(self, other: object) -> bool:
        """Maydonlari teng bo'lgan ikki alohida obyektni == bilan solishtirish."""
        if not isinstance(other, Book):
            return NotImplemented
        return (self.title, self.author, self.year) == (other.title, other.author, other.year)

    def __lt__(self, other: "Book") -> bool:
        """sorted() va .sort() uchun: kitoblarni nashr yili bo'yicha saralash."""
        if not isinstance(other, Book):
            return NotImplemented
        return self.year < other.year

    def __hash__(self) -> int:
        """
        TUZOT YECHIMI: Python'da __eq__ aniqlanganda, __hash__ avtomatik None bo'lib qoladi.
        Obyektni set yoki dict kaliti sifatida ishlatish uchun __hash__ ni aniqlash shart.
        """
        return hash((self.title, self.author, self.year))


# ==============================================================================
# 2. @dataclass bilan ancha ixcham yaratilgan muqobil klass
# ==============================================================================

@dataclass(frozen=True, order=True)
class DataClassBook:
    """
    @dataclass avtomatik ravishda __init__, __repr__, __eq__ metodlarini yaratadi.
    - order=True: __lt__, __le__, __gt__, __ge__ (saralash) metodlarini qo'shadi.
    - frozen=True: obyektni o'zgarmas (immutable) qiladi va avtomatik __hash__ yaratadi.
    """
    year: int  # Birinchi maydon bo'yicha saralaydi
    title: str
    author: str


# ==============================================================================
# 3. Demo va Testlar
# ==============================================================================

def main() -> None:
    print("=" * 60)
    print("1. QO'LDA YOZILGAN CLASS TESTLARI")
    print("=" * 60)

    b1 = Book("Python Asoslari", "Anvar Narzullayev", 2021)
    b2 = Book("Python Asoslari", "Anvar Narzullayev", 2021)
    b3 = Book("Toza Kod", "Robert Martin", 2008)
    b4 = Book("Algoritmlar", "Otabek", 2015)

    # __repr__ testi
    print(f"-> __repr__ ko'rinishi: {b1}")

    # __eq__ testi (tenglik)
    print(f"-> b1 == b2 (xotira adresi har xil, lekin ma'nosi teng): {b1 == b2}")
    print(f"-> b1 == b3: {b1 == b3}")

    # __lt__ va sorted() testi (saralash)
    book_list: List[Book] = [b1, b3, b4]
    sorted_books = sorted(book_list)
    print("\n-> Yili bo'yicha saralangan kitoblar:")
    for b in sorted_books:
        print(f"   {b.year} - {b.title}")

    # __hash__ testi (Set va Dict kaliti)
    print("\n--- Hash va Set testi ---")
    book_set: Set[Book] = {b1, b2, b3}
    print(f"-> Set ichidagi noyob elementlar soni (b1 va b2 bir xil bo'lgani uchun 2 ta bo'lishi kerak): {len(book_set)}")


    print("\n" + "=" * 60)
    print("2. DATACLASS TESTLARI")
    print("=" * 60)

    dc1 = DataClassBook(2021, "Python Asoslari", "Anvar Narzullayev")
    dc2 = DataClassBook(2008, "Toza Kod", "Robert Martin")

    print(f"-> Dataclass repr: {dc1}")
    print(f"-> Dataclass tenglik (dc1 == dc1): {dc1 == dc1}")
    print(f"-> Dataclass saralash (dc2 < dc1): {dc2 < dc1}")

    dc_set = {dc1, dc2}
    print(f"-> Dataclass (frozen=True) Set ichida ishlashi: {len(dc_set)} ta element")


if __name__ == "__main__":
    main()

```

---

### Tuzoq izohi (`__hash__` va `__eq__`):

* **Tuzoq:** Python-da agar siz klassda `__eq__` metodini yozsangiz, Python havfsizlik yuzasidan ushbu klassning `__hash__` metodini avtomatik ravishda `None` deb belgilaydi (`__hash__ = None`). Natijada obyektlarni `set` to'plamiga qo'shmoqchi bo'lsangiz yoki `dict` kaliti sifatida ishlatmoqchi bo'lsangiz, `TypeError: unhashable type` xatoligi kelib chiqadi.
* **Yechim:**
1. Oddiy klassda: Maydonlar qiymatidan iborat `tuple` yasab, uning `hash()` qiymatini qaytaruvchi `__hash__` metodini qo'lda yozish.
2. `@dataclass`da: `@dataclass(frozen=True)` parametri orqali obyektni o'zgarmas (immutable) deb belgilash. Bu avtomatik ravishda xavfsiz `__hash__` metodini generatsiya qilib beradi.
