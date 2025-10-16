BAŞLA

  GÜNLÜK_LIMIT ← 5000
  KULLANILAN_LIMIT ← 0
  PIN ← 1234
  HAK ← 3
  BAKIYE ← 10000

  // PIN DOĞRULAMA
  DÖNGÜ HAK > 0 İKEN
    YAZ "Lütfen PIN kodunuzu giriniz:"
    OKU GIRILEN_PIN

    EĞER GIRILEN_PIN = PIN İSE
       YAZ "PIN doğru. Hoşgeldiniz."
       ÇIK PIN_DOĞRULAMA_DÖNGÜSÜNDEN
    DEĞİLSE
       HAK ← HAK - 1
       YAZ "Hatalı PIN. Kalan deneme hakkı: ", HAK
    SON
  SON_DÖNGÜ

  EĞER HAK = 0 İSE
    YAZ "Kartınız bloke oldu. Lütfen bankanızla iletişime geçin."
    BİTİR
  SON

  // İŞLEM TEKRARI
  DÖNGÜ TRUE İKEN
    YAZ "Mevcut bakiyeniz: ", BAKIYE, " TL"
    YAZ "Günlük kullanılabilir limitiniz: ", GÜNLÜK_LIMIT - KULLANILAN_LIMIT, " TL"
    YAZ "Çekmek istediğiniz tutarı giriniz (20 TL katları olmalı):"
    OKU TUTAR

    // TUTAR KONTROLÜ
    EĞER TUTAR % 20 ≠ 0 İSE
      YAZ "Tutar 20 TL'nin katı olmalıdır."
    DEĞİLSE
      // BAKİYE KONTROLÜ
      EĞER TUTAR > BAKIYE İSE
        YAZ "Yetersiz bakiye."
      DEĞİLSE
        // GÜNLÜK LİMİT KONTROLÜ
        EĞER KULLANILAN_LIMIT + TUTAR > GÜNLÜK_LIMIT İSE
          YAZ "Günlük para çekme limitinizi aştınız."
        DEĞİLSE
          // İŞLEM ONAYI
          BAKIYE ← BAKIYE - TUTAR
          KULLANILAN_LIMIT ← KULLANILAN_LIMIT + TUTAR
          YAZ "İşleminiz başarıyla gerçekleşti."
          YAZ "Kalan bakiye: ", BAKIYE, " TL"
        SON
      SON
    SON

    // İŞLEM TEKRARI SEÇENEĞİ
    YAZ "Başka bir işlem yapmak istiyor musunuz? (E/H)"
    OKU CEVAP
    EĞER CEVAP = "H" İSE
      YAZ "Kartınızı almayı unutmayın. İyi günler!"
      ÇIK ANA_DÖNGÜDEN
    SON

  SON_DÖNGÜ

BİTİR
