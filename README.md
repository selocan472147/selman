import tkinter as tk
import random
import os

# Görevleri kaydedeceğimiz dosyanın adı
DOSYA_ADI = "gorevler.txt"

# --- Dosya İşlemleri Fonksiyonları ---

def gorevleri_yukle():
    """Görevleri dosyadan yükler, dosya yoksa boş liste döndürür."""
    gorevler_listesi = []
    if os.path.exists(DOSYA_ADI):
        with open(DOSYA_ADI, "r", encoding="utf-8") as file:
            for satir in file:
                gorevler_listesi.append(satir.strip())
    return gorevler_listesi

def gorevleri_kaydet(gorevler_listesi):
    """Güncel görev listesini dosyaya kaydeder."""
    with open(DOSYA_ADI, "w", encoding="utf-8") as file:
        for gorev in gorevler_listesi:
            file.write(gorev + "\n")

# --- Ana Pencere Ayarları ---
ana_pencere = tk.Tk()
ana_pencere.title("Tepki Ruleti (GUI)")
ana_pencere.geometry("500x750")
ana_pencere.resizable(False, False)

# --- Görev Listesi (Dosyadan Yüklenecek veya Başlangıç Görevleri) ---
gorevler = gorevleri_yukle()

# Eğer görev dosyası boşsa veya hiç görev yoksa, varsayılan görevleri ekle
if not gorevler:
    print("Mevcut görev bulunamadı. Başlangıç görevlerini ekliyorum.")
    gorevler = [
        "1 dakika boyunca kedi sesi çıkar",
        "Bir sonraki cümleni rap söyleyerek bitir",
        "Gülümseyerek 10 saniye ekrana bak",
        "En sevdiğin yemeği tarif et",
        "Rastgele bir kelime seç ve onunla ilgili kısa bir hikaye anlat",
        "Dans etmeye başla ve 15 saniye devam et",
        "Gizli bir yeteneğini göster",
        "Birine iltifat et",
        "30 saniye boyunca hiç durmadan konuş",
        "Şu anki ruh halini bir hayvan sesiyle anlat"
    ]
    gorevleri_kaydet(gorevler)

# --- Arayüz Fonksiyonları ---

def spin_wheel_effect(iterations_left, delay_ms, final_gorev, current_iteration=0):
    """
    Çarkın dönme efektini simüle eder.
    iterations_left: Kalan dönüş sayısı.
    delay_ms: Her adım arasındaki gecikme süresi (milisaniye).
    final_gorev: Animasyon sonunda gösterilecek asıl görev.
    current_iteration: Mevcut yineleme sayısı (dahili kullanım).
    """
    if iterations_left > 0:
        # Rastgele bir görev göster (dönme hissi için)
        temp_gorev = random.choice(gorevler)
        sonuc_etiket.config(text=f"Dönüyor... {temp_gorev}")
        sonuc_etiket.update_idletasks() # Ekranı hemen güncelle

        # Gecikmeyi ve bir sonraki adımı planla
        # delay_ms + 10 yaparak her adımda biraz daha yavaşlatırız (doğal durması için)
        ana_pencere.after(delay_ms, spin_wheel_effect, iterations_left - 1, delay_ms + 10, final_gorev, current_iteration + 1)
    else:
        # Animasyon bitti, asıl görevi göster
        sonuc_etiket.config(text=final_gorev)
        sonuc_etiket.config(fg="blue") # Rengi maviye geri çevir
        secim_butonu.config(state=tk.NORMAL) # Butonu tekrar aktif et
        guncel_gorev_sayisi_etiket.config(text=f"Toplam {len(gorevler)} görev var.")
        listbox_gorevleri_guncelle()

def gorev_sec_ve_goster(temizle_giris=False):
    """Rastgele bir görev seçer, çark dönme efektini başlatır."""
    if temizle_giris:
        yeni_gorev_giris.delete(0, tk.END)

    if not gorevler:
        sonuc_etiket.config(text="Hiç görev yok! Lütfen görev ekleyin.")
        return

    secim_butonu.config(state=tk.DISABLED) # Butonu pasif yap (dönme sırasında basılmasın)
    sonuc_etiket.config(fg="orange") # Metin rengini geçici olarak turuncu yap

    final_secilen_gorev = random.choice(gorevler) # Animasyon sonunda gösterilecek asıl görev

    # Çark dönme efektini başlat: 20 tekrar, 50 ms başlangıç gecikmesi
    spin_wheel_effect(iterations_left=20, delay_ms=50, final_gorev=final_secilen_gorev)


def gorev_ekle_gui():
    """Yeni görev giriş kutusundaki metni alıp görevi ekler."""
    yeni_gorev = yeni_gorev_giris.get().strip()
    if yeni_gorev:
        gorevler.append(yeni_gorev)
        gorevleri_kaydet(gorevler)
        yeni_gorev_giris.delete(0, tk.END)
        guncel_gorev_sayisi_etiket.config(text=f"Toplam {len(gorevler)} görev var.")
        sonuc_etiket.config(text=f"'{yeni_gorev}' görevi eklendi!")
        listbox_gorevleri_guncelle()
    else:
        sonuc_etiket.config(text="Lütfen eklenecek bir görev yazın!")

def listbox_gorevleri_guncelle():
    """Görevleri Listbox'ta (liste kutusu) günceller."""
    gorev_listesi_kutusu.delete(0, tk.END)
    for gorev in gorevler:
        gorev_listesi_kutusu.insert(tk.END, gorev)

def gorev_sil_gui():
    """Seçili görevi listeden ve dosyadan siler."""
    secili_indexler = gorev_listesi_kutusu.curselection()
    if not secili_indexler:
        sonuc_etiket.config(text="Lütfen silmek için bir görev seçin!")
        return

    for index in sorted(secili_indexler, reverse=True):
        silinen_gorev = gorevler.pop(index)
        sonuc_etiket.config(text=f"'{silinen_gorev}' görevi silindi.")

    gorevleri_kaydet(gorevler)
    guncel_gorev_sayisi_etiket.config(text=f"Toplam {len(gorevler)} görev var.")
    listbox_gorevleri_guncelle()

def gorev_duzenle_gui():
    """Seçili görevi düzenlemek için giriş kutusuna metnini yazar ve butonu günceller."""
    secili_indexler = gorev_listesi_kutusu.curselection()
    if not secili_indexler:
        sonuc_etiket.config(text="Lütfen düzenlemek için bir görev seçin!")
        return

    secili_index = secili_indexler[0]
    eski_gorev = gorevler[secili_index]

    yeni_gorev_giris.delete(0, tk.END)
    yeni_gorev_giris.insert(0, eski_gorev)

    ekle_butonu.config(text="Güncelle", command=lambda: gorev_guncelle_kaydet(secili_index))
    sonuc_etiket.config(text=f"'{eski_gorev}' görevini düzenleyebilirsiniz.")

def gorev_guncelle_kaydet(index_to_update):
    """Düzenlenmiş görevi kaydeder ve arayüzü günceller."""
    yeni_gorev_metni = yeni_gorev_giris.get().strip()
    if yeni_gorev_metni:
        gorevler[index_to_update] = yeni_gorev_metni
        gorevleri_kaydet(gorevler)
        yeni_gorev_giris.delete(0, tk.END)
        listbox_gorevleri_guncelle()
        sonuc_etiket.config(text=f"Görev '{yeni_gorev_metni}' olarak güncellendi!")
        ekle_butonu.config(text="Ekle", command=gorev_ekle_gui)
    else:
        sonuc_etiket.config(text="Görev boş bırakılamaz!")
        ekle_butonu.config(text="Ekle", command=gorev_ekle_gui)

# --- Görsel Bileşenlerin Tanımlanması ve Konumlandırılması ---

# Ana başlık etiketi
baslik_etiket = tk.Label(ana_pencere, text="Tepki Ruleti'ne Hoş Geldiniz!", font=("Arial", 18, "bold"))
baslik_etiket.pack(pady=15)

# Seçilen görevi veya bilgilendirme mesajlarını gösterecek etiket
sonuc_etiket = tk.Label(ana_pencere, text="Görev seçmek için butona tıkla!", font=("Arial", 14), wraplength=450, fg="blue")
sonuc_etiket.pack(pady=10)

# Yeni Görev Seç butonu
secim_butonu = tk.Button(ana_pencere, text="Yeni Görev Seç", command=lambda: gorev_sec_ve_goster(True), font=("Arial", 16), bg="#8BC34A", fg="white", activebackground="#689F38")
secim_butonu.pack(pady=20)

# Toplam görev sayısını gösteren etiket
guncel_gorev_sayisi_etiket = tk.Label(ana_pencere, text=f"Toplam {len(gorevler)} görev var.", font=("Arial", 10), fg="gray")
guncel_gorev_sayisi_etiket.pack(pady=5)

# --- Yeni Görev Ekleme Bölümü (FRAME içinde GRID ile yerleştirilmiştir) ---
# DİKKAT: Bu 'ekle_frame' tanımı ve pack çağrısı KODDA SADECE BİR KEZ OLMALI!
ekle_frame = tk.Frame(ana_pencere, bg="lightblue", bd=2, relief="solid") # Çerçeve: Açık mavi arka plan, kenarlık
ekle_frame.pack(pady=10)

# Grid sisteminde her sütuna genişlik verme
ekle_frame.grid_columnconfigure(0, weight=1)
ekle_frame.grid_columnconfigure(1, weight=3)
ekle_frame.grid_columnconfigure(2, weight=1)

# "Yeni Görev Ekle:" etiketi
gorev_ekle_etiket = tk.Label(ekle_frame, text="Yeni Görev Ekle:", font=("Arial", 12), bg="red", fg="white")
gorev_ekle_etiket.grid(row=0, column=0, padx=10, pady=10, sticky=tk.W + tk.E)

# Yeni görev giriş kutusu (Entry)
yeni_gorev_giris = tk.Entry(ekle_frame, width=40, font=("Arial", 12), bg="yellow")
yeni_gorev_giris.grid(row=0, column=1, padx=10, pady=10, sticky=tk.EW)

# Yeni Görev Ekle butonu (Yeşil renkli)
ekle_butonu = tk.Button(ekle_frame,
                        text="Ekle",
                        command=gorev_ekle_gui,
                        font=("Arial", 12, "bold"),
                        bg="darkgreen",
                        fg="white",
                        width=10,
                        height=2,
                        relief=tk.RAISED,
                        activebackground="#388E3C",
                        activeforeground="white")
ekle_butonu.grid(row=0, column=2, padx=10, pady=10, sticky=tk.W + tk.E)

# --- Görevleri Listeleme Bölümü ---
gorev_listesi_etiket = tk.Label(ana_pencere, text="Mevcut Görevler:", font=("Arial", 14, "bold"))
gorev_listesi_etiket.pack(pady=10)

# Listbox ve kaydırma çubuğunu içeren çerçeve
listbox_frame = tk.Frame(ana_pencere)
listbox_frame.pack()

# Kaydırma çubuğu (Listbox için)
scrollbar = tk.Scrollbar(listbox_frame, orient=tk.VERTICAL)
# Görev listesi kutusu (Listbox)
gorev_listesi_kutusu = tk.Listbox(listbox_frame, width=60, height=8, font=("Arial", 10), yscrollcommand=scrollbar.set, selectmode=tk.SINGLE)
scrollbar.config(command=gorev_listesi_kutusu.yview)

gorev_listesi_kutusu.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

# --- Yönetim Butonları (Sil ve Düzenle) ---
yonetim_frame = tk.Frame(ana_pencere)
yonetim_frame.pack(pady=10)

# Seçili Görevi Sil butonu (Kırmızı renkli)
sil_butonu = tk.Button(yonetim_frame, text="Seçili Görevi Sil", command=gorev_sil_gui, font=("Arial", 12), bg="#E53935", fg="white", activebackground="#C62828")
sil_butonu.pack(side=tk.LEFT, padx=10)

# Seçili Görevi Düzenle butonu (Turuncu renkli)
duzenle_butonu = tk.Button(yonetim_frame, text="Seçili Görevi Düzenle", command=gorev_duzenle_gui, font=("Arial", 12), bg="#FFA000", fg="white", activebackground="#FB8C00")
duzenle_butonu.pack(side=tk.LEFT, padx=10)

# --- Uygulama Başlangıcı ---
gorev_sec_ve_goster()
listbox_gorevleri_guncelle()

ana_pencere.mainloop()
