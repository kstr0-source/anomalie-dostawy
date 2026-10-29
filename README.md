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

## 0. Stan rozwiązania

Funkcje M1–M5 są uzupełnione. Oba notatniki zawierają wykonane komórki, wyniki, wykresy i odpowiedzi. Oryginalne testy: 10 passed, bez pominięć. Zachowano wersje bibliotek z dostarczonego requirements.txt. Weryfikacja lokalna: Python 3.12; dostarczone konfiguracje GitHub Actions i Render przewidują Python 3.13.

Notatniki otwieraj z katalogu projektu. Do ich ponownego wykonania w JupyterLab wybierz Restart Kernel and Run All Cells; środowisko notebooków instalujesz osobno, np. python -m pip install jupyterlab ipykernel.

Dashboard ocenia wybrany miesiąc przy stałym budżecie kontroli. Walidacja z zamrożonymi progami ze stycznia znajduje się w lab5, Z3.

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


## 5. Wyniki: styczeń, budżet 60 alarmów

Koszt = 25 zł × liczba alarmów + 800 zł × liczba przeoczonych anomalii. Kolumny etykieta i typ_anomalii nie są cechami i nie służą do wyboru progów. „IQR koszt” przy stałym budżecie to ranking według kosztu, zgodnie ze szkieletem; nie należy go mylić z klasycznym progiem Q3 + 1,5·IQR (73 alarmy w styczniu).

| Metoda | Alarmy | TP | FP | FN | Precision | Recall | F1 | Koszt [zł] |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| IQR koszt | 60 | 22 | 38 | 23 | 0.3667 | 0.4889 | 0.4190 | 19900 |
| Model normy | 60 | 45 | 15 | 0 | 0.7500 | 1.0000 | 0.8571 | 1500 |
| K-means | 60 | 17 | 43 | 28 | 0.2833 | 0.3778 | 0.3238 | 23900 |
| Isolation Forest | 60 | 20 | 40 | 25 | 0.3333 | 0.4444 | 0.3810 | 21500 |
| LOF | 60 | 37 | 23 | 8 | 0.6167 | 0.8222 | 0.7048 | 7900 |

## 6. Wyniki: luty, budżet 60 alarmów

Modele, skaler i parametry normy pochodzą ze stycznia. Z lutego wybieramy wyłącznie 60 najwyższych wyników, bez użycia etykiet. To wariant porównawczy przy stałym budżecie, odpowiadający działaniu dashboardu.

| Metoda | Alarmy | TP | FP | FN | Precision | Recall | F1 | Koszt [zł] |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| IQR koszt | 60 | 19 | 41 | 26 | 0.3167 | 0.4222 | 0.3619 | 22300 |
| Model normy | 60 | 45 | 15 | 0 | 0.7500 | 1.0000 | 0.8571 | 1500 |
| K-means | 60 | 11 | 49 | 34 | 0.1833 | 0.2444 | 0.2095 | 28700 |
| Isolation Forest | 60 | 9 | 51 | 36 | 0.1500 | 0.2000 | 0.1714 | 30300 |
| LOF | 60 | 35 | 25 | 10 | 0.5833 | 0.7778 | 0.6667 | 9500 |

## 7. Walidacja z Z3: luty, zamrożone progi ze stycznia

Próg każdej metody to wynik 60. dostawy w styczniu. Stosujemy konwencję szkieletu `alarm_prog(wynik, prog - 1e-9)`. Liczba alarmów w nowym miesiącu może różnić się od 60. Nie dopasowujemy ponownie modeli ani skalera. LOF na styczniu używa negative_outlier_factor_, a na nowych danych trybu novelty zgodnie ze szkieletem.

| Metoda | Alarmy | TP | FP | FN | Precision | Recall | F1 | Koszt [zł] |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| IQR koszt | 62 | 19 | 43 | 26 | 0.3065 | 0.4222 | 0.3551 | 22350 |
| Model normy | 57 | 45 | 12 | 0 | 0.7895 | 1.0000 | 0.8824 | 1425 |
| K-means | 68 | 13 | 55 | 32 | 0.1912 | 0.2889 | 0.2301 | 27300 |
| Isolation Forest | 64 | 10 | 54 | 35 | 0.1562 | 0.2222 | 0.1835 | 29600 |
| LOF | 51 | 34 | 17 | 11 | 0.6667 | 0.7556 | 0.7083 | 10075 |

## 8. Trzy wnioski ekonomiczne

1. W styczniu model normy przy 60 alarmach wykrywa 45 anomalii i kosztuje 1500 zł. Ranking kosztu wykrywa 22 i kosztuje 19 900 zł: model oszczędza 18 400 zł przy tej samej liczbie kontroli. Do miesięcznej kolejki kontroli wybieramy model normy.
2. W lutym próg modelu normy ustalony ze stycznia daje 57 alarmów, zero przeoczeń i koszt 1425 zł. Stały próg z > 3,5 daje 112 alarmów, również zero przeoczeń i koszt 2800 zł. Próg ze styczniowego budżetu ogranicza pracę o 55 kontroli i koszt o 1375 zł. Pozostawiamy go jako wariant operacyjny, monitorując skuteczność w kolejnych miesiącach; luty jest tu oceną, nie źródłem progu.
3. Po zmianie cennika od 16.03 stary model daje 218 alarmów w drugiej części marca (15,57 dziennie), koszt 5450 zł. Ujęcie dodatku ok. 0,69 zł/km zmniejsza to do 77 alarmów (5,50 dziennie) i 1925 zł, przy tych samych 24 wykrytych anomaliach. To o 141 kontroli i 3525 zł mniej. Po potwierdzonej zmianie taryfy aktualizujemy normę, zamiast traktować wszystkie odchylenia jako błędy dostaw.

## 9. Ograniczenie i interpretacja

Dane są syntetyczne i zawierają zawsze trzy znane typy anomalii po 15 rekordów; wynik 100% recall nie dowodzi takiej skuteczności na rzeczywistych fakturach ani innych typach błędów. Koszty 25 i 800 zł są założeniami, nie zmierzonymi stratami przedsiębiorstwa.

Minimum krzywej kosztu (np. model normy: 45 alarmów, 1125 zł w styczniu) wykorzystuje etykiety i jest tylko retrospektywnym punktem odniesienia, a nie procedurą wyboru progu. Aktualizacja modelu na całym marcu jest również demonstracją retrospektywną: wymaga sprawdzenia na późniejszym, niezależnym okresie. Mediana opóźnienia dnia jest dostępna po zebraniu dostaw z tego dnia, więc analiza działa wsadowo.

## 10. Publikacja i pliki wynikowe

Konfiguracje do publikacji znajdują się w `.github/workflows/testy.yml` i `render.yaml`. Repozytorium oraz aplikacja nie zostały opublikowane w ramach tego rozwiązania; po publikacji dopisz tutaj rzeczywiste adresy. Lokalny wynik testów nie oznacza jeszcze zielonego wyniku GitHub Actions.

- `wyniki_styczen.csv`: pełny ranking 1500 dostaw z lab4 Z6.
- `metryki_styczen_budzet60.csv`: styczeń, 60 alarmów.
- `metryki_luty_budzet60.csv`: luty, 60 alarmów.
- `metryki_luty_progi_styczen.csv`: walidacja lutowa z progami ze stycznia.
