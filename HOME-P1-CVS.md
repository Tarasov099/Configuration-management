import csv
import io

class CsvVirtualFile:
    """
    Представляет отдельную запись (строку) из CSV-файла как "виртуальный файл".
    """
    def __init__(self, headers, row):
        self.headers = headers
        self.row = row
        # Создаем уникальное имя для нашего "файла" на основе id
        self.name = f"item_{self.row.get('id', 'unknown')}.csv"

    def read(self):
        """
        Возвращает содержимое "файла" в виде строки, имитируя CSV-файл.
        """
        output = io.StringIO()
        writer = csv.DictWriter(output, fieldnames=self.headers)
        writer.writeheader()
        writer.writerow(self.row)
        return output.getvalue()

    def __str__(self):
        return self.name

class CsvVirtualFileSystem:
    """
    Виртуальная файловая система, источником которой является CSV-файл.
    """
    def __init__(self, csv_filepath):
        self.csv_filepath = csv_filepath
        self.data = []
        self.headers = []
        self._load_csv()

    def _load_csv(self):
        """
        Загружает данные из CSV-файла и сохраняет их.
        """
        try:
            with open(self.csv_filepath, mode='r', encoding='utf-8') as infile:
                reader = csv.DictReader(infile)
                self.headers = reader.fieldnames
                for row in reader:
                    self.data.append(row)
        except FileNotFoundError:
            print(f"Ошибка: Файл {self.csv_filepath} не найден.")
            self.headers = []
            self.data = []
        except Exception as e:
            print(f"Ошибка при чтении CSV: {e}")
            self.headers = []
            self.data = []

    def list_files(self):
        """
        Возвращает список имен всех "виртуальных файлов" (записей).
        """
        if not self.headers or not self.data:
            return []
        return [f"item_{row.get('id', 'unknown')}.csv" for row in self.data]

    def get_file(self, filename):
        """
        Возвращает объект CsvVirtualFile по имени файла.
        """
        if not self.headers or not self.data:
            return None

        # Извлекаем id из имени файла, предполагая формат "item_ID.csv"
        try:
            item_id = filename.split('_')[1].split('.')[0]
        except IndexError:
            print(f"Ошибка: Некорректный формат имени файла '{filename}'. Ожидается 'item_ID.csv'.")
            return None

        for row in self.data:
            if row.get('id') == item_id:
                return CsvVirtualFile(self.headers, row)
        print(f"Файл '{filename}' не найден.")
        return None

    def read_file(self, filename):
        """
        Удобный метод для чтения содержимого "файла" по имени.
        """
        vfs_file = self.get_file(filename)
        if vfs_file:
            return vfs_file.read()
        return None

# --- Пример использования ---

# 1. Создаем CSV-файл для примера
csv_content = """id,name,value,description
1,item_a,100,First item
2,item_b,200,Second item
3,item_c,150,Third item
"""
with open("data.csv", "w", encoding="utf-8") as f:
    f.write(csv_content)

# 2. Инициализируем нашу VFS
vfs = CsvVirtualFileSystem("data.csv")

# 3. Выводим список доступных "файлов"
print("Доступные файлы:")
for filename in vfs.list_files():
    print(f"- {filename}")

print("\n" + "="*20 + "\n")

# 4. Читаем содержимое конкретного "файла"
filename_to_read = "item_2.csv"
print(f"Чтение содержимого файла '{filename_to_read}':")
content = vfs.read_file(filename_to_read)
if content:
    print(content)

print("\n" + "="*20 + "\n")

# 5. Получаем объект "файла" и работаем с ним
filename_to_get = "item_1.csv"
print(f"Получение объекта файла '{filename_to_get}':")
vfs_file_obj = vfs.get_file(filename_to_get)
if vfs_file_obj:
    print(f"Имя файла: {vfs_file_obj}")
    print("Содержимое (через объект):")
    print(vfs_file_obj.read())

print("\n" + "="*20 + "\n")

# 6. Попытка прочитать несуществующий файл
print("Попытка прочитать несуществующий файл 'item_99.csv':")
non_existent_content = vfs.read_file("item_99.csv")
if not non_existent_content:
    print("Файл не найден (как и ожидалось).")

print("\n" + "="*20 + "\n")

# 7. Попытка получить файл с некорректным именем
print("Попытка получить файл с некорректным именем 'wrong_name.txt':")
vfs.get_file("wrong_name.txt")

ПРИМЕР CVS 
id,name,value,description
1,item_a,100,First item
2,item_b,200,Second item
3,item_c,150,Third item
