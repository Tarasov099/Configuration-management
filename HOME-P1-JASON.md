Пример структуры JSON
Допустим, у нас есть JSON-файл config.json со следующим содержимым:
```
json

{
  "app_settings": {
    "version": "1.2.0",
    "theme": "dark",
    "notifications": {
      "enabled": true,
      "level": "info"
    }
  },
  "user_profiles": [
    {
      "id": "user_101",
      "username": "alice",
      "email": "alice@example.com"
    },
    {
      "id": "user_102",
      "username": "bob",
      "email": "bob@example.com"
    }
  ]
}
```
Мы хотим создать VFS, где:

Структура JSON (ключи и вложенные объекты) будет отражать структуру каталогов.
Отдельные элементы массива (например, объекты пользователей) будут представлены как “файлы”.
Реализация на Python
python

import json
import os

class JsonVirtualFile:
    """
    Представляет собой отдельный элемент (например, объект в массиве JSON)
    как "виртуальный файл".
    """
    def __init__(self, file_path, content):
        self.path = file_path  # Полный путь в VFS
        self.content = content # Собственно данные, которые будут представлены как содержимое файла

    def read(self):
        """
        Возвращает содержимое "файла" в виде JSON-строки.
        """
        return json.dumps(self.content, indent=2)

    def __str__(self):
        return self.path

class JsonVirtualFileSystem:
    """
    Виртуальная файловая система, источником которой является JSON-файл.
    """
    def __init__(self, json_filepath):
        self.json_filepath = json_filepath
        self.root_data = {}
        self._load_json()

    def _load_json(self):
        """
        Загружает данные из JSON-файла.
        """
        try:
            with open(self.json_filepath, mode='r', encoding='utf-8') as infile:
                self.root_data = json.load(infile)
        except FileNotFoundError:
            print(f"Ошибка: Файл {self.json_filepath} не найден.")
            self.root_data = {}
        except json.JSONDecodeError:
            print(f"Ошибка: Некорректный формат JSON в файле {self.json_filepath}.")
            self.root_data = {}
        except Exception as e:
            print(f"Неизвестная ошибка при чтении JSON: {e}")
            self.root_data = {}

    def _resolve_path(self, path):
        """
        Рекурсивно обходит структуру данных JSON по заданному пути.
        Возвращает найденные данные или None.
        """
        parts = path.strip('/').split('/')
        current_data = self.root_data
        found_item = None

        for i, part in enumerate(parts):
            if not isinstance(current_data, dict):
                return None # Не можем идти дальше по пути

            if part in current_data:
                current_data = current_data[part]
                if i == len(parts) - 1: # Это последний элемент пути
                    found_item = current_data
            else:
                return None # Часть пути не найдена

        return found_item

    def list_items(self, path="/"):
        """
        Возвращает список элементов (файлов и каталогов) по заданному пути.
        """
        resolved_data = self._resolve_path(path)
        if resolved_data is None:
            print(f"Путь '{path}' не найден.")
            return []

        items = []
        if isinstance(resolved_data, dict):
            for key, value in resolved_data.items():
                item_path = os.path.join(path, key).replace("\\", "/")
                if isinstance(value, (dict, list)):
                    items.append(f"{item_path}/") # Это "каталог"
                else:
                    items.append(item_path) # Это "файл"
        elif isinstance(resolved_data, list):
            # Для списков, мы будем представлять каждый элемент как "файл"
            for i, item in enumerate(resolved_data):
                # Используем индекс как часть имени файла
                # Для более "человекочитаемых" имен можно было бы использовать другой механизм
                file_name = f"item_{i}.json" if not isinstance(item, (dict, list)) else f"item_{i}/"
                item_path = os.path.join(path, file_name).replace("\\", "/")
                items.append(item_path)
        return items

    def get_item(self, path):
        """
        Возвращает объект (JsonVirtualFile или dict/list) по заданному пути.
        Для "файлов" (не dict/list) возвращает JsonVirtualFile.
        """
        resolved_data = self._resolve_path(path)
        if resolved_data is None:
            print(f"Элемент по пути '{path}' не найден.")
            return None

        # Если это простой тип (не словарь и не список), считаем это "файлом"
        if not isinstance(resolved_data, (dict, list)):
            return JsonVirtualFile(path, resolved_data)
        else:
            # Если это словарь или список, возвращаем его как есть (представляет "каталог" или "массив")
            return resolved_data

    def read_file(self, path):
        """
        Удобный метод для чтения содержимого "файла" по пути.
        """
        item = self.get_item(path)
        if isinstance(item, JsonVirtualFile):
            return item.read()
        elif isinstance(item, (dict, list)):
            # Если это словарь или список, возвращаем его как JSON-строку
            return json.dumps(item, indent=2)
        else:
            return None

# --- Пример использования ---

# 1. Создаем JSON-файл для примера

```json
{
  "app_settings": {
    "version": "1.2.0",
    "theme": "dark",
    "notifications": {
      "enabled": true,
      "level": "info"
    }
  },
  "user_profiles": [
    {
      "id": "user_101",
      "username": "alice",
      "email": "alice@example.com"
    },
    {
      "id": "user_102",
      "username": "bob",
      "email": "bob@example.com"
    }
  ]
}
with open("config.json", "w", encoding="utf-8") as f:
    f.write(json_content)
```

# 2. Инициализируем нашу VFS
vfs = JsonVirtualFileSystem("config.json")

# 3. Выводим содержимое корневого каталога
print("Содержимое корневого каталога '/' :")
for item in vfs.list_items("/"):
    print(f"- {item}")

print("\n" + "="*30 + "\n")

# 4. Выводим содержимое подкаталога
print("Содержимое '/app_settings/' :")
for item in vfs.list_items("/app_settings/"):
    print(f"- {item}")

print("\n" + "="*30 + "\n")

# 5. Выводим содержимое массива (представленного как "каталог" с "файлами")
print("Содержимое '/user_profiles/' :")
for item in vfs.list_items("/user_profiles/"):
    print(f"- {item}")

print("\n" + "="*30 + "\n")

# 6. Читаем содержимое "файла" (простой элемент)
version_file_path = "/app_settings/version"
print(f"Чтение файла '{version_file_path}':")
content = vfs.read_file(version_file_path)
if content:
    print(content)

print("\n" + "="*30 + "\n")

# 7. Читаем содержимое "файла" (объект из массива)
user_alice_path = "/user_profiles/item_0.json" # Предполагается, что item_0.json представляет первый элемент массива
print(f"Чтение файла '{user_alice_path}':")
content = vfs.read_file(user_alice_path)
if content:
    print(content)

print("\n" + "="*30 + "\n")

# 8. Читаем содержимое "каталога" (объект из JSON)
settings_path = "/app_settings"
print(f"Чтение содержимого '{settings_path}' (как JSON):")
content = vfs.read_file(settings_path)
if content:
    print(content)

print("\n" + "="*30 + "\n")

# 9. Попытка прочитать несуществующий путь
print("Попытка прочитать несуществующий путь '/non_existent/item':")
non_existent_content = vfs.read_file("/non_existent/item")
if not non_existent_content:
    print("Элемент не найден (как и ожидалось).")
