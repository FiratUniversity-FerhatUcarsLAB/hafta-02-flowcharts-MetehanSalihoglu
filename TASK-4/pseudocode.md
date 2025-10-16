BAŞLA

  // Gerekli değişkenlerin tanımlanması
  TANIMLA ogrenci, seciliDerslerListesi = [], toplamKredi = 0
  TANIMLA KREDI_LIMITI = 35
  TANIMLA DANISMAN_ONAY_GPA_SINIRI = 2.5
  TANIMLA kayitDongusuAktif = true

  // 1. Öğrenci Girişi
  İŞLEM OgrenciGirisi()
    YAZ "Öğrenci No giriniz:"
    OKU ogrNo
    YAZ "Şifre giriniz:"
    OKU sifre

    ogrenci = VeritabanindanKullaniciDogrula(ogrNo, sifre)

    EĞER ogrenci BULUNAMADI İSE
      YAZ "Hatalı giriş bilgileri."
      BITIR
    DEĞİLSE
      YAZ "Giriş başarılı. Hoş geldin, " + ogrenci.ad
    SON_EĞER
  SON_İŞLEM

  // 2. Ders Seçim Döngüsü
  DÖNGÜ SÜRESİNCE (kayitDongusuAktif == true)
    YAZ "========================================="
    YAZ "AÇILAN DERSLER LİSTESİ:"
    GÖSTER VeritabanindanDersleriGetir()

    YAZ "\nSEÇİLEN DERSLER:"
    EĞER seciliDerslerListesi BOŞ İSE
      YAZ "Henüz ders seçmediniz."
    DEĞİLSE
      GÖSTER seciliDerslerListesi
    SON_EĞER
    YAZ "Toplam Kredi: " + toplamKredi

    YAZ "\nİşlem Seçiniz: (1: Ders Ekle, 2: Ders Çıkar, 3: Kaydı Tamamla)"
    OKU secim

    SEÇİME GÖRE (secim)
      DURUM 1: // Ders Ekle
        YAZ "Eklemek istediğiniz dersin kodunu giriniz:"
        OKU dersKodu
        secilecekDers = VeritabanindanDersBul(dersKodu)

        // --- KONTROL MEKANİZMALARI BAŞLANGICI ---
        // Kontrol 1: Kontenjan
        EĞER secilecekDers.mevcutOgrenci >= secilecekDers.kontenjan İSE
          YAZ "HATA: Ders kontenjanı dolu."
        DEĞİLSE
          // Kontrol 2: Ön Koşul
          EĞER secilecekDers.onKosulVar MI VE ogrenci.TranskriptteOnKosulBasariliMi(secilecekDers.onKosul) == false İSE
            YAZ "HATA: Bu dersin ön koşulunu sağlamıyorsunuz."
          DEĞİLSE
            // Kontrol 3: Zaman Çakışması
            EĞER ZamanCakismasiVarMi(secilecekDers, seciliDerslerListesi) == true İSE
              YAZ "HATA: Seçtiğiniz başka bir dersle zaman çakışması var."
            DEĞİLSE
              // Kontrol 4: Kredi Limiti
              EĞER (toplamKredi + secilecekDers.kredi) > KREDI_LIMITI İSE
                YAZ "HATA: Bu dersi eklerseniz kredi limitini aşıyorsunuz."
              DEĞİLSE
                // Tüm kontroller başarılı
                seciliDerslerListesi.EKLE(secilecekDers)
                toplamKredi = toplamKredi + secilecekDers.kredi
                YAZ "BAŞARILI: " + secilecekDers.ad + " dersi eklendi."
              SON_EĞER
            SON_EĞER
          SON_EĞER
        SON_EĞER
        // --- KONTROL MEKANİZMALARI BİTİŞİ ---
        KIR

      DURUM 2: // Ders Çıkar
        YAZ "Çıkarmak istediğiniz dersin kodunu giriniz:"
        OKU dersKodu
        cikarilacakDers = seciliDerslerListesi.Bul(dersKodu)
        EĞER cikarilacakDers BULUNDU İSE
          seciliDerslerListesi.CIKAR(cikarilacakDers)
          toplamKredi = toplamKredi - cikarilacakDers.kredi
          YAZ "BAŞARILI: " + cikarilacakDers.ad + " dersi çıkarıldı."
        DEĞİLSE
          YAZ "HATA: Bu ders zaten seçili değil."
        SON_EĞER
        KIR

      DURUM 3: // Kaydı Tamamla
        kayitDongusuAktif = false
        KIR
    SON_SEÇİM
  SON_DÖNGÜ

  // 3. Kayıt Özeti ve Onaylama
  YAZ "\n========================================="
  YAZ "KAYIT ÖZETİ"
  GÖSTER seciliDerslerListesi
  YAZ "Toplam Kredi: " + toplamKredi
  
  YAZ "Kaydı onaylıyor musunuz? (Evet/Hayır)"
  OKU onayCevabi

  EĞER onayCevabi == "Evet" İSE
    // Kontrol 5: Danışman Onayı
    EĞER ogrenci.gpa < DANISMAN_ONAY_GPA_SINIRI İSE
      YAZ "Not ortalamanız düşük olduğu için kaydınız danışman onayına gönderilmiştir."
      KaydiVeritabaninaYaz(ogrenci, seciliDerslerListesi, "Onay Bekliyor")
    DEĞİLSE
      YAZ "Ders kaydınız başarıyla tamamlanmıştır."
      KaydiVeritabaninaYaz(ogrenci, seciliDerslerListesi, "Onaylandı")
    SON_EĞER
  DEĞİLSE
    YAZ "Kayıt işlemi iptal edildi."
  SON_EĞER

SON