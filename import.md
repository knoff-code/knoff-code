```Родитель;Код;Наименование;Артикул;НаименованиеХарактеристики;ЕдиницаИзмерения;МетодОценки;НаправлениеДеятельности;КатегорияНоменклатуры;Поставщик;Склад;СпособПополнения;СрокПополнения;СтавкаНДС;СчетУчетаЗапасов;СчетУчетаЗатрат;ТипНоменклатуры;ЦеноваяГруппа;ИспользоватьХарактеристики;ИспользоватьПартии;Комментарий;СрокИсполненияЗаказа;НормаВремени;ФиксированнаяСтоимость;СтранаПроисхождения
Инструмент;PK-37813;"PROHARDEN ProKit 3/8""";PK-37813;"Набор 13шт, Cr-V сталь";шт;FIFO;Продажа;Ручной инструмент;;Основной склад;Заказ;3;20%;Товары;Товары;Запас;"Наборы инструментов";Да;Нет;"Набор головок 3/8"". Размеры: 8,9,10,11,12,13,14,15,16,17,19мм. В комплекте трещотка и удлинитель.";5;0;Нет;```

Для импорта товаров в 1С через Python есть несколько основных подходов:

## 1. Использование COM-объекта 1С

```python
import win32com.client

def import_products_via_com(products_data):
    """Импорт товаров через COM-объект 1С"""
    try:
        # Подключение к 1С
        v8 = win32com.client.Dispatch("V83.COMConnector")
        connection = v8.Connect("Srvr=сервер;Ref=база_данных;")
        
        # Получаем менеджер документов
        document_manager = connection.Документы.НовыйДокумент("ПоступлениеТоваровУслуг")
        
        for product in products_data:
            # Создаем строку документа
            line = document_manager.Товары.Добавить()
            line.Номенклатура = product['name']
            line.Количество = product['quantity']
            line.Цена = product['price']
            
        # Записываем документ
        document_manager.Записать()
        return True
        
    except Exception as e:
        print(f"Ошибка: {e}")
        return False

# Пример данных
products = [
    {'name': 'Товар 1', 'quantity': 10, 'price': 1000},
    {'name': 'Товар 2', 'quantity': 5, 'price': 1500}
]

import_products_via_com(products)
```

## 2. Импорт через XML/JSON файлы

```python
import xml.etree.ElementTree as ET
import json
from datetime import datetime

def create_1c_xml_import(products_data, filename):
    """Создание XML файла для импорта в 1С"""
    
    root = ET.Element("КоммерческаяИнформация")
    root.set("ВерсияСхемы", "2.04")
    root.set("ДатаФормирования", datetime.now().strftime("%Y-%m-%d"))
    
    for product in products_data:
        product_elem = ET.SubElement(root, "Товар")
        
        ET.SubElement(product_elem, "Ид").text = product.get('id', '')
        ET.SubElement(product_elem, "Наименование").text = product['name']
        ET.SubElement(product_elem, "Артикул").text = product.get('article', '')
        ET.SubElement(product_elem, "БазоваяЕдиница").text = product.get('unit', 'шт')
        
        # Цены
        prices_elem = ET.SubElement(product_elem, "Цены")
        price_elem = ET.SubElement(prices_elem, "Цена")
        ET.SubElement(price_elem, "ЦенаЗаЕдиницу").text = str(product['price'])
        ET.SubElement(price_elem, "Валюта").text = "руб"
    
    tree = ET.ElementTree(root)
    tree.write(filename, encoding="utf-8", xml_declaration=True)

# Создание XML файла
products_data = [
    {'id': '1', 'name': 'Товар 1', 'article': 'ART001', 'price': 1000},
    {'id': '2', 'name': 'Товар 2', 'article': 'ART002', 'price': 1500}
]

create_1c_xml_import(products_data, "import_products.xml")
```

## 3. Использование REST API (если в 1С настроен веб-сервис)

```python
import requests
import json

def import_via_rest_api(products_data, base_url, auth_token):
    """Импорт через REST API 1С"""
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {auth_token}'
    }
    
    for product in products_data:
        payload = {
            "Наименование": product['name'],
            "Артикул": product.get('article', ''),
            "Цена": product['price'],
            "ЕдиницаИзмерения": product.get('unit', 'шт')
        }
        
        response = requests.post(
            f"{base_url}/odata/standard.odata/Catalog_Номенклатура",
            headers=headers,
            json=payload,
            auth=('логин', 'пароль')  # если используется базовая аутентификация
        )
        
        if response.status_code == 201:
            print(f"Товар {product['name']} успешно импортирован")
        else:
            print(f"Ошибка импорта: {response.text}")

# Пример использования
products = [
    {'name': 'Товар 1', 'article': 'ART001', 'price': 1000},
    {'name': 'Товар 2', 'article': 'ART002', 'price': 1500}
]

import_via_rest_api(products, "http://1c-server/hrd", "your_auth_token")
```

## 4. Импорт через текстовые файлы (CSV)

```python
import csv
import os

def create_1c_csv_import(products_data, filename):
    """Создание CSV файла для импорта в 1С"""
    
    with open(filename, 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.writer(csvfile, delimiter=';')
        
        # Заголовок (зависит от конфигурации 1С)
        writer.writerow(['Наименование', 'Артикул', 'Цена', 'ЕдиницаИзмерения'])
        
        for product in products_data:
            writer.writerow([
                product['name'],
                product.get('article', ''),
                product['price'],
                product.get('unit', 'шт')
            ])

# Создание CSV файла
create_1c_csv_import(products_data, "import_products.csv")
```

## 5. Универсальная функция с выбором метода

```python
class ProductImporter:
    def __init__(self, method='xml', **kwargs):
        self.method = method
        self.connection_params = kwargs
    
    def import_products(self, products_data):
        if self.method == 'xml':
            return self._import_via_xml(products_data)
        elif self.method == 'com':
            return self._import_via_com(products_data)
        elif self.method == 'api':
            return self._import_via_api(products_data)
        elif self.method == 'csv':
            return self._import_via_csv(products_data)
        else:
            raise ValueError("Неизвестный метод импорта")
    
    def _import_via_xml(self, products_data):
        filename = self.connection_params.get('filename', 'import.xml')
        create_1c_xml_import(products_data, filename)
        return f"XML файл {filename} создан"
    
    def _import_via_com(self, products_data):
        return import_products_via_com(products_data)
    
    def _import_via_api(self, products_data):
        base_url = self.connection_params.get('base_url')
        token = self.connection_params.get('token')
        return import_via_rest_api(products_data, base_url, token)
    
    def _import_via_csv(self, products_data):
        filename = self.connection_params.get('filename', 'import.csv')
        create_1c_csv_import(products_data, filename)
        return f"CSV файл {filename} создан"

# Пример использования
importer = ProductImporter('xml', filename='products_import.xml')
result = importer.import_products(products_data)
print(result)
```

## Рекомендации по выбору метода:

1. **COM-объект** - для автоматизации на одном компьютере с 1С
2. **XML/JSON** - для регулярного обмена данными между системами
3. **REST API** - для веб-интеграции и работы по сети
4. **CSV** - простой способ для ручного импорта через обработки 1С

Выбор метода зависит от вашей конфигурации 1С, требований к безопасности и частоты обмена данными.
