## MoviesDataAnalyst
### 1. Film Arama ve Detaylı Bilgileri Görüntüleme
import pandas as pd

# CSV dosyasını oku
veri = pd.read_csv('filmtv_movies.csv')

# Excel dosyasına çevir ve kaydet
excel_adı = 'filmtv_movies.xlsx'  
veri.to_excel(excel_adı, index=False)

# Excel dosyasını oku
df = pd.read_excel('filmtv_movies.xlsx')

# Kullanıcıdan film adını al
film_adi = input("Film adını girin: ")

# Girilen film adına sahip veriyi filtrele
film_verileri = df[df['title'] == film_adi]

# Film verisi boşsa bulunamadı mesajını yazdır, değilse detayları göster
if film_verileri.empty:
    print(f"{film_adi} adlı film veri setinde bulunamadı.")
else:
    # Film detaylarını ekrana yazdır
    print("\nFilm Bilgileri:")
    print("Film ID: ", film_verileri['filmtv_id'].values[0])
    print("Yıl: ", film_verileri['year'].values[0])
    # Diğer bilgiler de aynı şekilde devam eder...
#### Bu kod bloğu, kullanıcının bir film adı girmesini sağlar, ardından bu film adına sahip veriyi filtreler ve eğer bulunursa detaylı bilgilerini ekrana yazdırır.
### 2. Benzer Filmleri Bulma ve Karşılaştırma
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.metrics.pairwise import cosine_similarity
from sklearn.feature_extraction.text import CountVectorizer

# Excel dosyasını oku
df = pd.read_excel('filmtv_movies.xlsx')

# Kullanıcıdan film adını al
film_adi = input("Film adını girin: ")

# Girilen film adına sahip veriyi filtrele
film_verileri = df[df['title'] == film_adi]

# Eğer film verisi boşsa bulunamadı mesajını yazdır, değilse benzer filmleri bul ve karşılaştır
if film_verileri.empty:
    print(f"{film_adi} adlı film veri setinde bulunamadı.")
else:
    # Seçilen özellikleri al
    selected_features = film_verileri[['avg_vote', 'country', 'genre']]

    # Veri setine yeni sütunlar ekle
    df['selected_avg_vote'] = selected_features['avg_vote'].values[0]
    df['selected_country'] = selected_features['country'].values[0]
    df['selected_genre'] = selected_features['genre'].values[0]

    # CountVectorizer kullanarak benzerlik matrisini oluştur
    vectorizer = CountVectorizer()
    feature_matrix = vectorizer.fit_transform(df[['title', 'selected_avg_vote', 'selected_country', 'selected_genre']].astype('U').agg(' '.join, axis=1))

    # Cosine similarity matrisini hesapla
    cosine_sim = cosine_similarity(feature_matrix, feature_matrix)

    # Kullanıcının girdiği film ile diğer filmler arasındaki benzerlikleri bul
    similarities = cosine_sim[film_verileri.index.values[0]]

    # Benzerlik skorlarına göre sırala
    similar_movies_indices = np.argsort(similarities)[::-1]

    # En yakın 5 filmin bilgilerini ve grafikleri yazdır
    print("\nEn Yakın 5 Film:")
    for index in similar_movies_indices[1:6]:  # İlk film kendisi olduğu için 1'den başlıyoruz
        movie_info = df.iloc[index]
        # Film bilgilerini yazdır
        print(f"\nFilm ID: {movie_info['filmtv_id']}")
        print("Film Adı: ", movie_info['title'])
        print("Ortalama Oy: ", movie_info['avg_vote'])
        print("Ülke: ", movie_info['country'])
        print("Tür: ", movie_info['genre'])

        # Görselleştirme için bar plot
        plt.figure(figsize=(10, 6))
        sns.barplot(x=['Seçilen Film', f"Film {index - 1}"],
                    y=[selected_features['avg_vote'].values[0], movie_info['avg_vote']],
                    palette=['blue', 'green'])
        plt.title('Ortalama Oy Karşılaştırması')
        plt.ylabel('Ortalama Oy')
        plt.ylim(0, 10)  # 0 ile 10 arasında değer alır
        plt.show()
#### Bu kod bloğu, kullanıcının bir film adı girmesini sağlar, ardından bu film adına sahip veriyi filtreler. Seçilen filmle diğer filmler arasındaki benzerlikleri hesaplar, benzerlik skorlarına göre sıralar ve en yakın 5 filmi görsel ve metin olarak karşılaştırır.
### 3. Tür Dağılımı Grafiği
import pandas as pd
import matplotlib.pyplot as plt

# Excel dosyasını oku
df = pd.read_excel('filmtv_movies.xlsx')

# 'genre' sütununa göre grupla ve her türün filmlerinin sayısını al
genre_counts = df.groupby('genre').size()

# Toplam film sayısına göre yüzdelik oranları hesapla
genre_percentages = genre_counts / genre_counts.sum() * 100

# Bar plot oluştur
plt.figure(figsize=(12, 6))
genre_percentages.plot(kind='bar')
plt.title('Türlerin Film Yüzdelik Dağılımı')
plt.xlabel('Tür')
plt.ylabel('Yüzde (%)')
plt.show()

##### Bu kod bloğu, film verilerini türlerine göre gruplar, her türün filmlerinin sayısını hesaplar, yüzdelik oranları hesaplar ve bunları bir bar grafiği ile görselleştirir.
### 4. Effort Dağılımı Pasta Grafiği
sizes = df['effort'].value_counts()
labels = [f"Effort-{i}" for i in sizes.index]
colors = ["blue", "pink", "red", "yellow", "purple"]

plt.pie(sizes, labels=labels, colors=colors, autopct="%1.1f%%", startangle=90, wedgeprops=dict(width=0.4))
plt.axis("equal")
plt.title("Effort Dağılımını gösteren pasta modeli")
plt.show()
#### Bu kod bloğu, "effort" sütunundaki değerlere göre bir pasta grafiği oluşturur ve çeşitli effort seviyelerinin dağılımını görselleştirir.
### 5. Özelliklere Göre Ortalama Değerler Çatıplotu
import seaborn as sns

# 'humor', 'rhythm', 'tension', 'erotism' sütunlarındaki değerlere göre ortalama değerleri al
df_means = df[['humor', 'rhythm', 'tension', 'erotism']].mean()

# Gruplandırılmış veriyi çatıplot ile görselleştir
plt.figure(figsize=(12, 6))
ax = sns.catplot(data=df_means.reset_index(), x='index', y=0, kind='bar', aspect=1.25)
plt.xlabel('Özellik')
plt.ylabel('Ortalama Değer')
plt.title('Özelliklere Göre Ortalama Değerler')
plt.show()
#### Bu kod bloğu, "humor", "rhythm", "tension", "erotism" sütunlarındaki değerlere göre ortalama değerleri hesaplar ve bu özelliklerin ortalamalarını bir çatıplot ile görselleştirir.
