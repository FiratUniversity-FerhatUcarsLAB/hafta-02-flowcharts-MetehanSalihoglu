BAŞLA

    // 1. Kimlik Doğrulama
    GİRİŞ: TC_Kimlik, Şifre
    EĞER TC_Kimlik ve Şifre doğru İSE
        DEVAM ET
    DEĞİLSE
        MESAJ: "Kimlik doğrulama başarısız"
        BİTİR

    // 2. Poliklinik Seçimi
    POLIKLINIK_LISTESI ← sistemden getir
    GÖSTER: POLIKLINIK_LISTESI
    GİRİŞ: secilenPoliklinik

    // 3. Doktor Listesi
    DOKTOR_LISTESI ← secilenPoliklinik içindeki doktorlar
    GÖSTER: DOKTOR_LISTESI
    GİRİŞ: secilenDoktor

    // 4. Uygun Saatler
    SAAT_LISTESI ← secilenDoktor için uygun saatler
    GÖSTER: SAAT_LISTESI
    GİRİŞ: secilenSaat

    // 5. Randevu Onaylama
    MESAJ: "Seçilen Randevu: " + secilenPoliklinik + " - " + secilenDoktor + " - " + secilenSaat
    GİRİŞ: onay (EVET/HAYIR)
    
    EĞER onay = EVET İSE
        RANDEVU ← oluştur(secilenPoliklinik, secilenDoktor, secilenSaat, TC_Kimlik)
        MESAJ: "Randevunuz başarıyla oluşturuldu"
        
        // 6. SMS Gönderme
        SMS_GÖNDER(TC_Kimlik, RANDEVU)
        MESAJ: "Randevu bilgileri SMS ile gönderildi"
    DEĞİLSE
        MESAJ: "Randevu iptal edildi"
    
BİTİR
BAŞLA

  // 1. KİMLİK DOĞRULAMA
  İŞLEM KimlikDogrulama()
    YAZ "T.C. Kimlik Numaranızı giriniz:"
    OKU tcKimlikNo
    YAZ "Barkod veya Protokol Numaranızı giriniz:"
    OKU protokolNo

    // Kullanıcı bilgileri veritabanından doğrulanır
    kullanici = VeritabanindaKullaniciDogrula(tcKimlikNo, protokolNo)

    EĞER kullanici BULUNAMADI İSE
      YAZ "Hatalı kimlik numarası veya protokol numarası. Lütfen bilgilerinizi kontrol edip tekrar deneyin."
      DÖN BAŞA
    DEĞİLSE
      YAZ "Kimlik doğrulama başarılı."
      TahlilVarligiKontrolu(kullanici) // Bir sonraki adıma geç
    SON_EĞER
  SON_İŞLEM


  // 2. TAHLİL VARLIĞI KONTROLÜ
  İŞLEM TahlilVarligiKontrolu(kullanici)
    // Kullanıcıya ait tahlil kayıtları veritabanından sorgulanır
    tahlilKayitlari = VeritabanindanTahlilleriGetir(kullanici.id)

    EĞER tahlilKayitlari BOŞ İSE
      YAZ "Sisteme kayıtlı herhangi bir tahlil sonucunuz bulunmamaktadır."
      // Program sonlandırılır veya ana menüye dönülür
      BITIR
    DEĞİLSE
      SonucHazirMiKontrolu(tahlilKayitlari) // Bir sonraki adıma geç
    SON_EĞER
  SON_İŞLEM


  // 3. SONUÇ HAZIR MI KONTROLÜ
  İŞLEM SonucHazirMiKontrolu(tahlilKayitlari)
    // Tahlillerin durumu kontrol edilir (örneğin, 'Sonuçlandı', 'Beklemede')
    sonuclananTahliller = []
    bekleyenTahliller = []

    DÖNGÜ her tahlil İÇİN tahlilKayitlari
      EĞER tahlil.durum == "Sonuçlandı" İSE
        sonuclananTahliller.EKLE(tahlil)
      DEĞİLSE
        bekleyenTahliller.EKLE(tahlil)
      SON_EĞER
    SON_DÖNGÜ

    GoruntulemeVeyaBeklemeMesaji(sonuclananTahliller, bekleyenTahliller) // Bir sonraki adıma geç
  SON_İŞLEM


  // 4. GÖRÜNTÜLEME VEYA BEKLEME MESAJI
  İŞLEM GoruntulemeVeyaBeklemeMesaji(sonuclananTahliller, bekleyenTahliller)
    EĞER sonuclananTahliller BOŞ DEĞİLSE
      YAZ "Sonuçlanan Tahlilleriniz:"
      GÖSTER sonuclananTahliller // Tahlil adı, tarihi vb. bilgiler listelenir

      YAZ "Sonucunu görüntülemek istediğiniz tahlili seçiniz:"
      OKU secilenTahlil

      YAZ "Tahlil Sonucu Detayları:"
      GÖSTER secilenTahlil.detaylar // Sonuçlar ekrana yazdırılır
      
      PDFIndirme(secilenTahlil) // Bir sonraki adıma geç
    SON_EĞER

    EĞER bekleyenTahliller BOŞ DEĞİLSE
      YAZ "Sonucu Henüz Hazır Olmayan Tahlilleriniz:"
      GÖSTER bekleyenTahliller
      YAZ "Bu tahlillerin sonuçları çıktığında sistem üzerinden görüntüleyebilirsiniz."
    SON_EĞER

    EĞER sonuclananTahliller BOŞ VE bekleyenTahliller BOŞ İSE
      YAZ "Sistemde görüntülenecek veya bekleyen tahliliniz bulunmuyor."
    SON_EĞER
  SON_İŞLEM


  // 5. PDF İNDİRME
  İŞLEM PDFIndirme(tahlilSonucu)
    YAZ "Tahlil sonucunu PDF olarak indirmek ister misiniz? (Evet/Hayır)"
    OKU cevap

    EĞER cevap == "Evet" İSE
      // Tahlil sonucu verilerinden bir PDF dosyası oluşturulur
      pdfDosyasi = PDFOlustur(tahlilSonucu)
      
      // Oluşturulan PDF dosyası kullanıcıya indirilir
      IndirmeBaslat(pdfDosyasi)
      
      YAZ "Tahlil sonucunuz PDF olarak başarıyla indirildi."
    DEĞİLSE
      YAZ "İşlem sonlandırıldı."
    SON_EĞER
  SON_İŞLEM


  // PROGRAMIN BAŞLANGIÇ NOKTASI
  KimlikDogrulama()

SON