# CES-507-Proje
Güneş Enerjisi Projesi Mühendislik Fakültesi F Blok
# Bir kez: depoyu bilgisayarınıza alın
git clone https://[github.com/nazlicerenkara/ces507-fblok-gunes-nseb.git](https://github.com/nazlicerenkara/CES-507-)
cd ces507-fblok-gunes-nseb

# Her görev için: issue #3 "Güneş konumu fonksiyonları" olsun
git switch main
git pull
git switch -c feat/3-gunes-konumu

# Dosyaları oluşturup düzenleyin (aşağıdaki örnek), sonra:
git status                       # ne değişti?
git diff                         # satır satır kontrol
git add src/fblok/solar_geometry.py tests/test_solar_geometry.py
git commit -m "feat: Spencer ile deklinasyon ve zaman denklemi eklendi (#3)"

# GitHub'a gönderin
git push -u origin feat/3-gunes-konumu

# Pull Request açın (GitHub CLI ile ya da web arayüzünden)
gh pr create --fill --body "Closes #3. pvlib ile karşılaştırma testi eklendi."
"""Güneş geometrisi: Spencer (1971) yaklaşımları. Kaynak: Duffie & Beckman."""
import numpy as np

def _gun_acisi(gun_no):
    """B = 2π(n-1)/365 (radyan)."""
    return 2.0 * np.pi * (np.asarray(gun_no) - 1) / 365.0

def deklinasyon(gun_no):
    """Güneş deklinasyonu δ (radyan)."""
    b = _gun_acisi(gun_no)
    return (0.006918
            - 0.399912 * np.cos(b) + 0.070257 * np.sin(b)
            - 0.006758 * np.cos(2 * b) + 0.000907 * np.sin(2 * b)
            - 0.002697 * np.cos(3 * b) + 0.00148 * np.sin(3 * b))

def zaman_denklemi(gun_no):
    """Zaman denklemi E (dakika)."""
    b = _gun_acisi(gun_no)
    return 229.18 * (0.000075
                     + 0.001868 * np.cos(b) - 0.032077 * np.sin(b)
                     - 0.014615 * np.cos(2 * b) - 0.04089 * np.sin(2 * b))

import numpy as np
import pytest
from fblok.solar_geometry import deklinasyon, zaman_denklemi

def test_deklinasyon_gundonumleri():
    # Yaz gündönümü (n=172) ≈ +23,44°, kış gündönümü (n=355) ≈ -23,44°
    assert np.degrees(deklinasyon(172)) == pytest.approx(23.44, abs=0.3)
    assert np.degrees(deklinasyon(355)) == pytest.approx(-23.44, abs=0.3)

def test_deklinasyon_ekinoks():
    # Mart ekinoksu (n≈80) civarında δ ≈ 0
    assert np.degrees(deklinasyon(80)) == pytest.approx(0.0, abs=0.5)

def test_zaman_denklemi_kasim():
    # Kasım başında E ≈ +16 dakika (maksimum civarı)
    assert zaman_denklemi(307) == pytest.approx(16.4, abs=1.0)

def test_pvlib_ile_uyum():
    pvlib = pytest.importorskip("pvlib")
    n = np.arange(1, 366)
    fark = np.degrees(deklinasyon(n)) - np.degrees(
        pvlib.solarposition.declination_spencer71(n))
    assert np.max(np.abs(fark)) < 0.01
                    
