# ikea1
selling
#Web_scrapping_project
import tkinter as tk
from tkinter import Scrollbar
from bs4 import BeautifulSoup as bs
import requests
from tabulate import tabulate

def search_products():
    name = product_name_entry.get()
    headers = {
        "User-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36",
        "Accept-Language": "en-US,en;q=0.5"
    }
    result = []
    amazon(name, headers, result)
    flipkart(name, headers, result)
    snapdeal(name, headers, result)

    if not result:
        result_text.insert(tk.END, "Sorry, couldn't find items at this time.", 'error')
    else:
        result.sort(key=lambda x: float(x[1].split()[-1]))
        data = tabulate(result, headers=("Product", "Price", "Source"), tablefmt="grid", maxcolwidths=[80, None])
        result_text.insert(tk.END, data, 'success')
        with open("Product_List.txt", 'w', encoding="utf-8") as file:
            file.write(data)

def amazon(name, headers, result):
    url = r"https://www.amazon.in/s?k=" + name
    webpage = requests.get(url, headers=headers)
    soup = bs(webpage.text, "html.parser")
    links = soup.find_all('div', attrs={'class': "s-result-item"})
    for link in links:
        a = link.find('span', attrs={"class": "a-text-normal"})
        b = link.find('span', attrs={"class": "a-offscreen"})
        if a and b:
            result.append([a.text, b.text.replace(',', '').replace('₹', "Rs. "), "Amazon"])

def flipkart(name, headers, result):
    url = "https://www.flipkart.com/search?q=" + name
    webpage = requests.get(url, headers=headers)
    soup = bs(webpage.text, "html.parser")
    links = soup.find_all('div', attrs={'class': "_2kHMtA"})
    for link in links:
        a = link.find('div', attrs={'class': '_4rR01T'})
        b = link.find('div', attrs={'class': '_30jeq3 _1_WHN1'})
        if a and b:
            result.append([a.text, b.text.replace(',', '').replace('₹', "Rs. "), "Flipkart"])

def snapdeal(name, headers, result):
    url = "https://www.snapdeal.com/search?keyword=" + name
    webpage = requests.get(url, headers=headers)
    soup = bs(webpage.text, "html.parser")
    links = soup.find_all('div', attrs={'class': "product-desc-rating"})
    for link in links:
        a = link.find('p', attrs={'class': 'product-title'})
        b = link.find('span', attrs={'class': 'lfloat product-price'})
        if a and b:
            result.append([a.text, b.text.replace(',', '').replace('₹', "Rs. "), "Snapdeal"])

# GUI setup
window = tk.Tk()
window.title("Product Search")
window.geometry("800x600")

product_name_label = tk.Label(window, text="Enter Product Name You Want to Search:", fg="blue")
product_name_label.pack(pady=(10, 5))

product_name_entry = tk.Entry(window, width=50)
product_name_entry.pack(pady=(0, 10))

search_button = tk.Button(window, text="Search", command=search_products, bg="green", fg="white")
search_button.pack()

# Result text with scrollbars
result_frame = tk.Frame(window, bd=2, relief=tk.GROOVE)
result_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

result_text = tk.Text(result_frame, wrap=tk.WORD, width=80, height=20)
result_text.pack(fill=tk.BOTH, expand=True)

result_scrollbar_y = Scrollbar(result_frame, orient=tk.VERTICAL, command=result_text.yview)
result_scrollbar_y.pack(side=tk.RIGHT, fill=tk.Y)
result_text.config(yscrollcommand=result_scrollbar_y.set)

result_scrollbar_x = Scrollbar(window, orient=tk.HORIZONTAL, command=result_text.xview)
result_scrollbar_x.pack(side=tk.BOTTOM, fill=tk.X)
result_text.config(xscrollcommand=result_scrollbar_x.set)

window.mainloop() 

