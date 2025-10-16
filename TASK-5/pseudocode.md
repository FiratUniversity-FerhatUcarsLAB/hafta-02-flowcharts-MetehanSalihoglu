BAŞLA

  // Sistem durumu değişkeni
  TANIMLA sistemAktif = false // Başlangıçta sistem kapalı

  // Ana döngü - Cihaz çalıştığı sürece devam eder
  DÖNGÜ SÜRESİNCE (true)
  
    // ANA KONTROL: Sistem kurulu mu?
    EĞER sistemAktif == true İSE
    
      // Sensörlerden verileri oku
      hareketDurumu = OkuHareketSensoru()
      kapiDurumu = OkuKapiPencereSensoru()
      
      // ANOMALİ KONTROLÜ: Beklenmedik bir durum var mı?
      EĞER hareketDurumu == "HAREKET_VAR" VEYA kapiDurumu == "AÇIK" İSE
      
        YAZ "Anormallik tespit edildi. Alarm süreci başlatılıyor..."
        KamerayiAktifEt()
        
        // YANLIŞ ALARM KONTROLÜ: Ev sahibi evde mi?
        evSahibiEvdeMi = KonumServisiIleKontrolEt()
        
        EĞER evSahibiEvdeMi == false İSE
        
          // ALARM SEVİYESİ BELİRLEME
          alarmSeviyesi = 1 // Varsayılan düşük seviye
          olayMesaji = ""
          
          EĞER kapiDurumu == "AÇIK" İSE
            alarmSeviyesi = 3 // Kapı açılması yüksek öncelikli
            olayMesaji = "UYARI: Kapı/Pencere açıldı!"
          DEĞİLSE EĞER hareketDurumu == "HAREKET_VAR" İSE
            alarmSeviyesi = 2 // Hareket orta öncelikli
            olayMesaji = "UYARI: Ev içinde hareket algılandı!"
          SON_EĞER
          
          // BİLDİRİM GÖNDERME
          BildirimGonder("Uygulama", olayMesaji, alarmSeviyesi)
          BildirimGonder("SMS", olayMesaji, alarmSeviyesi)
          BildirimGonder("Email", olayMesaji, alarmSeviyesi)
          
          // BEKLEME VE KULLANICI TEPKİSİ DÖNGÜSÜ
          tepkiBekleniyor = true
          beklemeSayaci = 0
          DÖNGÜ SÜRESİNCE (tepkiBekleniyor == true VE beklemeSayaci < 60)
            BEKLE(1 saniye) // 1 saniye bekle
            kullaniciKomutu = OkuKullaniciKomutlarini()
            
            // ALARM SIFIRLAMA VEYA DEVAM ETTİRME
            EĞER kullaniciKomutu == "SIFIRLA" İSE
              AlarmSifirla()
              YAZ "Alarm kullanıcı tarafından sıfırlandı."
              tepkiBekleniyor = false
            DEĞİLSE EĞER kullaniciKomutu == "DEVAM_ET" İSE
              // Siren çalma gibi bir sonraki adıma geçilebilir
              YAZ "Kullanıcı alarmı onayladı. Sirenler aktif ediliyor."
              tepkiBekleniyor = false
            SON_EĞER
            
            beklemeSayaci = beklemeSayaci + 1
          SON_DÖNGÜ
          
          EĞER tepkiBekleniyor == true İSE // Zaman aşımına uğradıysa
            YAZ "Kullanıcıdan tepki gelmedi. Alarm devam ediyor."
            // Otomatik olarak siren çal veya güvenlik birimlerini haberdar et
          SON_EĞER
          
        DEĞİLSE // Ev sahibi evde, yanlış alarm
          YAZ "Ev sahibi evde. Yanlış alarm olabilir, düşük öncelikli bildirim gönderiliyor."
          BildirimGonder("Uygulama", "Sistem evde olduğunuzu algıladı, bir sensör tetiklendi.", 1)
        SON_EĞER
        
      SON_EĞER // Anormallik kontrolü sonu
      
    SON_EĞER // Sistem aktif mi kontrolü sonu
    
    // Ana döngünün çok hızlı çalışmasını önlemek için kısa bir bekleme
    BEKLE(2 saniye)
    
  SON_DÖNGÜ // Ana döngü sonu

SON
