# Wykrywanie anomalii w dostawach

Projekt zaliczeniowy z eksploracji danych (zajęcia 4–5). Dane są syntetyczne: 1500 dostaw miesięcznie, w każdym miesiącu 45 celowo zmodyfikowanych rekordów (`etykieta = 1`). Etykieta służy wyłącznie do oceny modeli i nigdy nie jest cechą.

## Zawartość

| Plik | Co to jest |
|---|---|
| `anomalie.py` | funkcje: reguły IQR, model normy kosztu, k-means, Isolation Forest, LOF, metryki, koszt, krzywa kosztu |
| `tests/test_anomalie.py` | testy automatyczne (`pytest -q`) |
| `app.py` | dashboard Streamlit |
| `dane/` | `dostawy_styczen.csv` (uczenie), `dostawy_luty.csv` (walidacja), `dostawy_marzec.csv` (nowy cennik od 16.03) |
| `requirements.txt` | biblioteki w wersjach, na których projekt sprawdzono |
| `render.yaml` | konfiguracja wdrożenia na Render |
| `.github/workflows/testy.yml` | testy uruchamiane automatycznie na GitHubie po każdym `git push` |

## 0. Najpierw: uzupełnij `anomalie.py`

W pliku `anomalie.py` dziewięć funkcji ma tylko opis i `raise NotImplementedError` (oznaczenia M1–M5 odpowiadają zadaniom przygotowawczym z karty zadań). Uzupełniaj je po kolei i uruchamiaj `pytest -q`: każdy zielony test to jedna działająca funkcja. Dashboard zadziała, gdy przejdą wszystkie testy.

## 1. Uruchomienie lokalne

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
pytest -q                 # wszystkie testy powinny przejść
streamlit run app.py      # otworzy się przeglądarka z dashboardem
```

## 2. GitHub

```bash
git init
git add .
git commit -m "Projekt: wykrywanie anomalii w dostawach"
git branch -M main
git remote add origin https://github.com/TWOJ_LOGIN/anomalie-dostawy.git
git push -u origin main
```

Repozytorium zakładasz wcześniej na github.com (New repository, bez README). Po wypchnięciu zakładka **Actions** pokaże wynik testów.

## 3. Render

1. Zaloguj się na render.com kontem GitHub.
2. **New → Blueprint**, wskaż repozytorium: Render odczyta `render.yaml`. Alternatywnie **New → Web Service** i wpisz ręcznie:
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `streamlit run app.py --server.port $PORT --server.address 0.0.0.0`
3. Po kilku minutach aplikacja będzie dostępna pod adresem `https://…onrender.com`. W planie bezpłatnym usługa usypia po okresie bezczynności; pierwsze wejście trwa wtedy dłużej.

Nazwy przycisków w GitHubie i Renderze mogą się różnić w zależności od wersji serwisów.

## 4. Co oddać

Adres repozytorium, adres aplikacji na Renderze, notatnik z porównaniem metod oraz w README: wyniki liczbowe (alarmy, precision, recall, F1, koszt) dla budżetu 60 alarmów, trzy wnioski ekonomiczne i jedno ograniczenie analizy.
