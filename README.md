# SCT_SD_4 
import requests
from bs4 import BeautifulSoup
import csv

# URL of the e-commerce website's product listing page
url = "http://example.com/products"

# Send a request to the website
response = requests.get(url)
response.raise_for_status()  # Ensure the request was successful

# Parse the HTML content of the page
soup = BeautifulSoup(response.text, 'html.parser')

# Find the container that holds product information
products = soup.find_all('div', class_='product-item')

# List to hold extracted product information
product_data = []

for product in products:
    name = product.find('h2', class_='product-name').text.strip()
    price = product.find('span', class_='product-price').text.strip()
    rating = product.find('span', class_='product-rating').text.strip()
    
    product_data.append([name, price, rating])

# Define the CSV file header
header = ['Name', 'Price', 'Rating']

# Write data to a CSV file
with open('products.csv', 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerow(header)
    writer.writerows(product_data)

print("Product information extracted and saved to products.csv")
