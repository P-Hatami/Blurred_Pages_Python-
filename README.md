# Blurred_Pages_Python-
# Recognizing blurred pages of a PDF. 
!pip install -q PyMuPDF
from google.colab import drive
import fitz
import cv2
import numpy as np

# اتصال به گوگل درایو
drive.mount('/content/drive')

def find_blurry_pages(pdf_path, blur_threshold=100.0):
    print(f"\nدر حال باز کردن فایل و شروع بررسی صفحات...")
    try:
        doc = fitz.open(pdf_path)
    except Exception as e:
        print(f"خطا در باز کردن فایل! مطمئن شوید آدرس را درست کپی کرده‌اید.\nجزئیات خطا: {e}")
        return

    blurry_pages = []
    total_pages = len(doc)
    print(f"تعداد کل صفحات فایل: {total_pages}\n")
    
    for page_num in range(total_pages):
        page = doc.load_page(page_num)
        pix = page.get_pixmap(matrix=fitz.Matrix(2, 2)) 
        img = np.frombuffer(pix.samples, dtype=np.uint8).reshape(pix.h, pix.w, pix.n)
        
        if pix.n == 4:
            gray = cv2.cvtColor(img, cv2.COLOR_BGRA2GRAY)
        elif pix.n == 3:
            gray = cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
        else:
            gray = img

        variance = cv2.Laplacian(gray, cv2.CV_64F).var()
        if variance < blur_threshold:
            blurry_pages.append(page_num + 1)
            print(f"⚠️ صفحه {page_num + 1} تار است. (امتیاز: {variance:.2f})")

    print("\n" + "="*40)
    if blurry_pages:
        print(f"نتیجه نهایی: صفحاتی که باید دوباره عکس بگیرید:\n{blurry_pages}")
    else:
        print("عالی! هیچ صفحه تاری پیدا نشد.")
    print("="*40)

# گرفتن آدرس فایل از شما
print("\n" + "-"*50)
file_path = input("لطفاً آدرس فایل را اینجا Paste کنید و اینتر بزنید: ")
find_blurry_pages(file_path.strip(), blur_threshold=100.0)
