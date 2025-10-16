BASLA  // Ana Süreç

BASLA  // Kullanıcı Girişi
    EGER kullanıcı_oturumu_var_mı 
        ISE kullanıcıyı_anasayfaya_yönlendir
    DEGILSE
        kullanıcıdan_giriş_bilgilerini_al
        EGER bilgiler_dogru_mu 
            ISE kullanıcı_oturumunu_baslat
        DEGILSE
            hata_mesajı_göster
            GİRİŞ_ADIMINI_TEKRARLA
        BITIR
    BITIR
BITIR  // Kullanıcı Girişi

BASLA  // Ürün Seçimi ve Sepete Ekleme
    KULLANICI ürün_listesini_gezin
    DONgU ürün_seçilene_kadar
        EGER ürün_stokta_var_mı 
            ISE ürünü_seçilen_adet_ile_sepete_ekle
                sepet_toplam_fiyatını_güncelle
        DEGILSE
            stok_yetersiz_mesajı_göster
        BITIR
    DONgU_SONU
BITIR  // Ürün Seçimi

BASLA  // Sepet Yönetimi
    DONgU kullanıcı_sepeti_görüntülüyor_veya_düzenliyor_iken
        kullanıcı_ürün_adedi_değiştirirse 
            EGER yeni_adet_stokta_var_mı 
                ISE adet_güncelle
                    sepet_toplam_fiyatını_güncelle
            DEGILSE
                stok_yetersiz_mesajı_göster
            BITIR
        kullanıcı_ürünü_silerse
            ürünü_sepetten_kaldır
            sepet_toplam_fiyatını_güncelle
        BITIR
    DONgU_SONU
BITIR  // Sepet Yönetimi

BASLA  // İndirim Kodu Uygulama
    kullanıcı_indirim_kodu_girer
    EGER kod_var_mı VE kod_geçerli_mi VE koşullar_uygun_mu
        ISE indirimi_hesapla
            sepet_toplam_fiyatından_indirimi_düş
    DEGILSE
        geçersiz_kod_mesajı_göster
    BITIR
BITIR  // İndirim Kodu

BASLA  // Kargo Hesaplama
    kullanıcı_adresini_girer_veya_seçer
    kargo_ücretini_hesapla(bölge, ağırlık, kampanya)
    sepet_toplam_fiyatına_kargo_ücretini_ekle
    tahmini_teslimat_süresini_göster
BITIR  // Kargo Hesaplama

BASLA  // Ödeme Aşaması
    kullanıcı_ödeme_yöntemini_seçer

    EGER yöntem_kredi_kartı 
        ISE kart_bilgilerini_al
            kart_bilgilerini_güvenli_şekilde_doğrula
            ödeme_sonucu ← ödeme_sağlayıcısına_gönder
            EGER ödeme_sonucu_başarılı 
                ISE sipariş_oluştur
                    stokları_güncelle
                    kullanıcıya_onay_maili_gönder
            DEGILSE
                ödeme_reddi_mesajı_göster
                yeniden_deneme_sun
            BITIR
    DEGILSE_EGER yöntem_kapıda_ödeme 
        ISE sipariş_oluştur
            stokları_güncelle
            kapıda_ödeme_bilgilendirmesi_gönder
    DEGILSE_EGER yöntem_havale 
        ISE sipariş_oluştur
            stokları_rezerve_et
            havale_talimatı_gönder
    BITIR
BITIR  // Ödeme Aşaması

BASLA  // Sipariş ve Kargo Takibi
    sipariş_veritabanına_kaydet
    kargo_sistemi_entegrasyonu_varsa
        kargo_kaydı_oluştur
    kullanıcıya_sipariş_numarasını_göster
    kargo_takip_linki_oluştur
BITIR  // Sipariş Takibi

BITIR  // Ana Süreç
