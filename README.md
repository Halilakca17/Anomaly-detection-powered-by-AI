#  Proje Açıklaması

Bu projede, çeşitli günlere ait ağ trafiği verileri birleştirilerek makine öğrenmesi ve derin öğrenme tabanlı bir **anomali tespit sistemi** geliştirilmiştir.

##  Kullanılan Veri Setleri

| Dosya Adı         | Boyut     | Saldırı Türü / Özellikler                      |
|------------------|-----------|------------------------------------------------|
| `03-01-2018.csv` | 107.84 MB | %28 oranında **infiltration** saldırısı       |
| `03-02-2018.csv` | 352.37 MB | %27 oranında **botnet** saldırısı             |
| `02-16-2018.csv` | 333.72 MB | Yoğunlukla **Hulk** DDoS saldırısı            |
| `02-21-2018.csv` | 328.89 MB | Yoğunlukla **HOIC** DDoS saldırısı            |
| `Özel Veri Seti` | -         | Kullanıcı tarafından oluşturulmuş örnek trafik verisi |

Tüm veri setleri birleştirilmiş ve gerekli ön işleme adımlarından geçirilmiştir. Eksik, sonsuz ve tekrarlı değerler temizlenmiş; sayısal özellikler normalize edilmiştir.

##  Kullanılan Modeller

Aşağıdaki modeller ile çok sınıflı saldırı tespiti gerçekleştirilmiştir:

-  **Random Forest**
-  **Derin Öğrenme** (Yapay Sinir Ağı)
-  **Support Vector Machine (SVM)**
-  **K-Nearest Neighbors (KNN)**

Modeller, eğitildikten sonra aşağıdaki metrikler ile değerlendirilmiştir:

- `Precision`
- `Recall`
- `F1-Score`
- `Confusion Matrix`

##  Projenin Amacı

Bu çalışmanın temel amacı, farklı saldırı türlerinin tespit edilebilirliğini karşılaştırmak ve en yüksek doğruluğu sağlayan yöntemi belirlemektir. Böylece, ağ trafiğindeki zararlı aktivitelerin daha etkili biçimde analiz edilmesi ve önlenmesi hedeflenmektedir.

---

*Not: Bu proje, [CICIDS 2018](https://www.unb.ca/cic/datasets/ids-2018.html) veri kümesine dayalı olarak gerçekleştirilmiştir.*

##  Kurulum

Bu projeyi çalıştırmak için aşağıdaki adımları izleyebilirsiniz:

1. Ortamı oluşturun (isteğe bağlı ama önerilir):

```bash
python -m venv env
source env/bin/activate    # Windows kullanıyorsan: .\env\Scripts\activate
```

2. Gerekli kütüphaneleri yükleyin:

```bash
pip install pandas numpy tensorflow matplotlib seaborn scikit-learn scipy
```

3. Veri dosyasını hazırlayın:
   - `final_dataset.csv` dosyasını projenin ana dizinine yerleştirin
   - Alternatif olarak, kod içerisindeki dosya yolunu güncelleyin

4. Kodu çalıştırın:

```bash
python ids.py
```

##  Veri Analizi Süreci

### 1. Veri Ön İşleme
- **Veri Birleştirme**: Farklı günlere ait CSV dosyaları birleştirildi
- **Örnekleme**: Her sınıf için maksimum 50,000 örnek alındı (dengesizliği önlemek için)
- **Temizlik**: NaN ve sonsuz değerler kaldırıldı
- **Tip Optimizasyonu**: Memory kullanımını azaltmak için veri tipleri optimize edildi

### 2. Keşifsel Veri Analizi (EDA)

####  Trafik Süresi Analizi
Bağlantı süreleri kategorik olarak analiz edildi:
- **Anlık (0s)**: Kesintisiz bağlantılar
- **Çok Kısa (0-1s)**: Hızlı işlemler
- **Kısa (1-10s)**: Normal web trafiği
- **Orta (10-60s)**: Dosya indirme/yükleme
- **Uzun (60s+)**: Uzun süreli bağlantılar

####  Port Analizi
- **En Çok Kullanılan Portlar**: Top 15 port listesi
- **Servis Portları**: HTTP (80), HTTPS (443), SSH (22), FTP (21) vb.
- **Port Aralık Dağılımı**:
- Well-known Portlar (0–1023):
- Sistem veya çekirdek servisler tarafından kullanılır.
- Örnekler: HTTP (80), HTTPS (443), DNS (53), SSH (22)

- Registered Portlar (1024–49151):
- Uygulama geliştiricileri tarafından kayıtlı hizmetlerde kullanılır.
- Örnekler: MySQL (3306), PostgreSQL (5432), Docker (2375)

- Dynamic/Private Portlar (49152–65535):
- Geçici bağlantılar için sistem tarafından dinamik olarak atanır.
- Genelde istemci tarafında bağlantı başlatırken kullanılır.
####  Protokol Analizi
- **TCP**: En yaygın protokol
- **UDP**: İkinci sırada
- **ICMP**: Ping ve tanı trafiği
- **Flag Kombinasyonları**: En yaygın TCP flag kombinasyonları

### 3. Anomali Tespiti
**Isolation Forest** algoritması kullanılarak:
- Isolation Forest, verideki anormallikleri yani olağan dışı ve sıradışı veri noktalarını tespit etmek için kullanılan etkili bir           algoritmadır. Temel prensibi, anormal verilerin normal verilere göre daha kolay izole edilebileceği varsayımıdır; bu yüzden rastgele  oluşturduğu ağaçlarla veriyi bölerek, hızlıca “sıradışı” olanları ortaya çıkarır.
- PCA ile 2D görselleştirme
- Anomali skorları dağılım analizi

### 4. İstatistiksel Analizler
- **Korelasyon Matrisi**: Özellikler arası ilişkiler
- **Feature Importance**: Random Forest ile özellik önemleri
- **Sınıf Bazlı Karşılaştırma**: Benign vs Malicious trafik analizi

## Model Mimarileri

### Derin Öğrenme Modeli (TensorFlow)
```python
Architecture:
├── Input Layer: Veri boyutuna göre otomatik
├── Dense Layer 1: 128 nöron + ReLU + Dropout(0.3)
├── Dense Layer 2: 64 nöron + ReLU + Dropout(0.3)
└── Output Layer: Sınıf sayısı + Softmax

Hyperparameters:
├── Epochs: 3
├── Batch Size: 256
├── Optimizer: Adam
├── Loss: Sparse Categorical Crossentropy
└── GPU Support: Otomatik tespit
```

### Diğer Modeller
- **Random Forest**: 100 ağaç, varsayılan parametreler
- **SVM**: RBF kernel, varsayılan parametreler
- **KNN**: 5 komşu, varsayılan parametreler


### Görselleştirmeler
-  Confusion Matrix heatmap'leri
-  Feature importance grafikleri
-  Sınıf dağılım grafikleri  
-  Korelasyon heatmap'i
-  Anomali skorları dağılımı

## En Önemli Özellikler

Model analizi sonucunda tespit edilen kritik özellikler:

1. **Init Fwd Win Byts**: Bağlantının başında göndericinin açtığı pencere büyüklüğü. Normalden büyük veya küçük olması, bağlantıda anormallik veya saldırı belirtisi olabilir.
2. **Dst Port**: Trafiğin yönlendiği hedef port numarası. Bazı portlar (örneğin 80, 443) sık kullanılır; alışılmadık portlar kötü amaçlı etkinlik gösterebilir.
3. **Fwd Seg Size Min**: Gönderilen en küçük TCP segment boyutu. Anormal değerler, paket manipülasyonu veya saldırı habercisi olabilir.
4. **Fwd Header Len**: Paket başlık uzunluğu. Sıradışı uzunluklar, protokol hatası veya kötü niyetli müdahale işareti olabilir.
5. **Bwd Pkts/s**: Geri yöndeki paket sayısı saniyede. Normalden yüksekse, ağda olağandışı hareketlilik veya saldırı olabilir.  



### Model Hiperparametrelerini Değiştirme
```python
# Random Forest için
RandomForestClassifier(
    n_estimators=200,  # Ağaç sayısını artır
    max_depth=10,      # Maksimum derinlik
    random_state=42
)

# Neural Network için
model.add(Dense(256, activation='relu'))  # Daha fazla nöron
model.compile(optimizer='adamax')         # Farklı optimizer
```


## 📊 Örnek Çıktılar

### Model Performans Karşılaştırması
```
Model Accuracy Comparison:
├── Random Forest: 0.9507
├── Neural Network: 0.9450
├── SVM: 0.9484
└── KNN: 0.9493
```

##  Sonuç ve Değerlendirme

Bu projede, farklı günlerden elde edilen çeşitli ağ trafiği verileri birleştirilerek çok sınıflı saldırı tespiti amacıyla hem makine öğrenmesi hem de derin öğrenme modelleri uygulanmıştır. Gerçekleştirilen analizler ve karşılaştırmalar sonucunda şu bulgulara ulaşılmıştır:

-  **Random Forest** modeli, %95.07 doğruluk oranı ile en yüksek başarıyı sağlamıştır.
-  **Derin Öğrenme (Yapay Sinir Ağı)** modeli, %94.50 doğruluk ile karmaşık örüntüleri tanımada etkili olmuştur.
-  **SVM** ve 🔍 **KNN** modelleri de benzer başarı oranları göstermiştir ancak eğitim süresi ve veri ölçeği açısından bazı sınırlamalar gözlemlenmiştir.
- Özellik önem dereceleri incelendiğinde, `Init Fwd Win Byts`, `Dst Port`, `Fwd Header Len`, `Fwd Seg Size Min`, ve `Bwd Pkts/s` gibi ağ trafiğine ait teknik parametrelerin saldırıların tespitinde kritik rol oynadığı görülmüştür.

Yapılan çalışmalar, farklı saldırı türlerinin ağ trafiği üzerindeki davranışsal izlerini ortaya koymuş ve bu izlerin başarılı bir şekilde sınıflandırılabileceğini göstermiştir.

---

###  Gelecek Çalışmalar İçin Öneriler

-  **Özellik mühendisliği** daha da geliştirilebilir (örneğin zamansal özetler, oturum bazlı analizler vb.).
-  **Derin öğrenme modelleri** için daha uzun eğitim süresi ve hiperparametre optimizasyonu ile performans artırılabilir.
-  **Gerçek zamanlı ağ trafiği** üzerinde test edilerek modellerin pratikteki başarıları değerlendirilebilir.
-  **Transfer learning** veya önceden eğitilmiş modeller ile daha az veriyle daha hızlı sonuçlar elde edilebilir.

>  Not: Bu projenin çıktıları, siber güvenlik alanında anomali tespiti sistemlerinin etkinliğini artırmaya yönelik somut bir temel sunmaktadır.



##  Katkıda Bulunma

1. Fork yapın
2. Feature branch oluşturun (`git checkout -b feature/amazing-feature`)
3. Commit yapın (`git commit -m 'Add amazing feature'`)
4. Branch'i push edin (`git push origin feature/amazing-feature`)
5. Pull Request oluşturun
