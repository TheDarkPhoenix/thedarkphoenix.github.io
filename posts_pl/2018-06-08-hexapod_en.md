---
layout: post
title: Hexapod
categories:
  - Robots
excerpt: |
  <img width="200" height="200" src="/pics/11_hexapod/minihexi.jpg">
---

A project I was involved in for the first 2 years of my studies as part of the Integra student research group. My contribution was software for robot control and stereovision.

{% include figure.html image="/pics/11_hexapod/hexi.jpg" width="600" height="800" %}

To move robot uses 18 Hitec servos, 3 for each leg. We used a Raspberry Pi 2 as the robot's main computer, it communicated with the Pololu servo controller via a UART interface, which set servo positions. The entire design was powered by a 4000 mAh LiPol battery, which allowed the robot to be used for about 1h. We supplied voltage from the battery using voltage converters (3 to power the servos and 1 for the Raspberry). We also used an ADC MCP3008 to measure voltage on the battery. It communicated with the Raspberry via SPI, and then the voltage was sent and displayed in the client application.

{% include googleDrivePlayer.html id="1kR9fSGo-6mLM5SdvNPUtoAvgJeeI8Lu-/preview" %}

The robot is controlled from a laptop, which communicates with the Raspberry via WiFi via TCP protocol. Both a pad and a keyboard can be used to move the robot. In order to properly control it, I first wrote an application with a mathematical model of the robot. Inverse kinematics was implemented on this model, on which I then created models of the robot's movement. The obtained angles were then converted into PWM signals and sent to servos through controller.

{% include figure.html image="/pics/11_hexapod/symulacja.png" width="600" height="800" %}

## Controlling the robot

The application running on the laptop contains visualizations of the robot model and receives control signals from the user, which it then sent to the program on the Raspberry. It makes the appropriate conversions to PWM signals, which are sent to the servo controller. Feedback is also sent from the Raspberry to the user application with the current voltage on the battery, so that its level is monitored in real time. A separate program is used to implement stereovision. The Raspberry sends images from both cameras using the gstreamer. On the laptop, both images are received and then using the appropriate matrices (obtained during calibration) the disparity image matrix is transformed and calculated. I also applied a filter after the disparity conversion, which further improves the results.

Hexapod model applications can work in 2 modes:

#### Model mode

In model mode, you can move around the environment using the W/S/A/D/Q/E keys and the alpha, beta and gamma bars for changing the viewing angle. The robot model can be moved using the w/s/a/d/q/e keys and the numbers used to select the appropriate robot walking mode.

{% include googleDrivePlayer.html id="1ixW05vog_nNaR9UR6ZqT5rZBkIWlBJeN/preview" %}

#### Hexapod connection mode

It runs if you additionally pass the IP address of the Raspberry to the program at startup. The control of the application on the laptop does not change. The difference with the previously discussed mode (model mode) is that now the corresponding commands are also sent to the Hexapod. Additionally the voltage on the battery is also displayed on the laptop screen.

A more detailed description of the elements of the realized application

#### GUI

Zadaniem najbardziej oddalonym od samej idei Hexapoda był moduł wyświetlania, czyli GUI. Do wyświetlania użyto wyłącznie prostego okienkowego trybu wyświetlania dołączonego do biblioteki OpenCV w celu debugowania. Zastosowałem rzutowanie perspektywiczne zgodnie ze wzorami zawartymi w [1](https://en.wikipedia.org/wiki/3D_projection "1"), aby otrzymać symulację w 3D. Zdefiniowałem płaszczyznę kamery, na którą odbywało się rzutowanie świata 3D symulacji. Dodałęm także przesuwanie płaszczyzną kamery za pomocą przycisków oraz możliwość jej obracania przy pomocy toolbarów.

#### Robot

To jest główny moduł odpowiedzialny za obliczenia związane ze sterowaniem robota. Zawarłem w nim implementację kinematyki odwrotnej w podejściu trygonometrycznym opisaną w [2](https://oscarliang.com/inverse-kinematics-and-trigonometry-basics/ "2"). Główna klasa robota zawiera klasę nóg robota, które są wydzielone. Jest osobna klasa, w której znajdują się różne algorytmy chodzenia.

#### Chodzenie

Najlepsze efekty dawało chodzenie po paraboli. Dobierane są kolejne punkty paraboli, która zaczyna się w miejscu, gdzie aktualnie znajduje się końcówka nogi robota. Kończy się tam, gdzie ma się ostatecznie znaleźć. Przy użyciu kinematyki odwrotnej wyliczane są kąty tak, aby końcówka danej nogi znalazła się w punkcie docelowym.

#### Pozostałe moduły

Kontroler, który odpowiednio interpretuje wysłane komendy na funkcje, np.: chodzenia lub poruszania bazą robota. Znajduje się tutaj również moduł do komunikacji z serwami (tylko w wersji programu dla Raspberry). Zamieniane są wyliczone kąty dla każdej nogi na odpowiednie sygnały PWM, dla każdego serwa (jedna noga składa się z trzech serwonapędów). Następnie są one wysyłane.

#### TCP

W module TCP znajduję się cały kod do komunikacji pomiędzy Raspberry a laptopem. Kod został zaczerpnięty z [3](https://github.com/vichargrave/tcpsockets "3"). W ramach komunikacji laptop łączy się z Raspberry i na porcie 8081 odbiera aktualne napięcie baterii, które Raspberry stale udostępnia. Komunikacja w ramach odczytu napięcia baterii odbywa się w osobnym wątku, aby nie zakłócać pozostałych operacji. Natomiast na port 8080 aplikacja z laptopa wysyła podane komendy.

{% include button.html text="Aplikacja na laptopa" icon="github" link="https://github.com/macstepien/HexapodPC" color="#0366d6" %}

{% include button.html text="Aplikacja na Raspberry" icon="github" link="https://github.com/macstepien/HexapodRaspberry" color="#0366d6" %}

## Stereowizja

Poniżej przedstawie krótki opis etapów realizacji stereowizji:

### Wykonanie zdjęć kalibracyjnych

Aby skalibrować parę stereowizyjną wykonywane są zdjęcia ustalonego wzoru, np. szachownicy. W tym celu napisałem aplikację, która wyświetlała obraz z obu kamer. Co ustalony czas zapisywała zdjęcie pod odpowiednią nazwą oraz tworzyła listę utworzonych obrazów. Wykonałem około 50 zdjęć tak, aby jak najlepiej pokryć cały obszar widoku kamer.

{% include figure.html image="/pics/11_hexapod/stereo12.png" width="600" height="800" caption="Przykładowa para zdjęć wykonanych podczas kalibracji"%}

Widoczny na zdjęciach zamieszczonych powyżej jest także następny napotkany problem - obiektywy w obudowach kamer zamontowane były nierówno. Oczywiście to przesunięcie eliminowane jest podczas kalibracji, jednak w rezultacie podczas właściwej stereowizji ograniczone jest pole widzenia od góry i dołu (część pixeli widoczna jest tylko przez jedną kamerkę).

### Kalibracja

Do samej kalibracji użyłem przykładu z książki Learning OpenCV. W parametrach wywołania umieszczano wysokość oraz szerokość szachownicy (liczba pól), a także długość boku kwadratu w centymetrach. Bardzo ważne jest, aby szachownica miała kwadratowe pola, gdyż przy użyciu szachownicy o bokach różniących się nieznacznie cały proces kalibracji zostałby przeprowadzony niepoprawnie wraz z listą zdjęć, powstałych w trakcie tej błędnej kalibracji.

{% include figure.html image="/pics/11_hexapod/stereo3.png" width="400" height="800" caption="Wynik wykrywania krawędzi szachownicy"%}

{% include figure.html image="/pics/11_hexapod/stereo4.png" width="700" height="800" caption="Wynik rektyfikacji" %}

W wyniku działania kalibracji otrzymałem parametry zewnętrzne oraz wewnętrzne kamer, które następnie używane były do właściwej stereowizji. Uzyskano błędy:
błąd średniokwadratowy (RMS) = 0.0503053
średni błąd epipolarny = 0.517146
Są to wartości niskie, co świadczy o poprawnie wykonanej kalibracji.

### Wybór algorytmu oraz dobranie parametrów

Najpierw przetestowałem dostępne w bibliotece OpenCV algorytmy BM i SGBM, lepsze rezultaty uzyskałem dla SGBM. Jest on bardziej wymagający obliczeniowo, aczkolwiek przy rozdzielczości 320x240 można było uzyskać dobre przetwarzanie w czasie rzeczywistym. Następnie dobrałem optymalne parametry dla tego algorytmu. W tym celu napisałem aplikację na podstawie przykładu użycia SGBM zawartego w bibliotece OpenCV. Dodałem do niej suwak tak, aby można było wygodnie zmieniać parametry i obserwować uzyskiwane rezultatu.

{% include figure.html image="/pics/11_hexapod/stereo5.png" width="400" height="800" caption="Widok okna z paskami do zmiany parametrów" %}

{% include figure.html image="/pics/11_hexapod/stereo6.png" width="400" height="800" caption="Przykładowa scena użyta do doboru parametrów" %}

{% include figure.html image="/pics/11_hexapod/stereo7.png" width="400" height="800" caption="Wynik przeprowadzenia stereowizji" %}

### Dobranie parametrów dla postfiltracji

W celu uzyskania gładszego obrazu stereowizyjnego zastosowałem postfiltrację, która dodana została stosunkowo niedawno do modułów dodatkowych biblioteki OpenCV - contrib.

{% include figure.html image="/pics/11_hexapod/stereo9.png" width="400" height="800" caption="Efekt przeprowadzenia algorytmu SGBM przy optymalnych parametrach" %}

{% include figure.html image="/pics/11_hexapod/stereo10.png" width="400" height="800" caption="Efekt przeprowadzenia postfiltracji" %}

### Ostateczna aplikacja

W ostatecznej aplikacji połączone zostały wszystkie opisane elementy. W wyniku czego otrzymano przesył obrazu z obu kamer na laptopa. Tam obraz był rektyfikowany, a pixele były dopasowywane za pomocą algorytmu SGBM. Przy użyciu posfiltracji obraz dysparcji jest poprawiany i na koniec wyświetlany. Dalszym etapem rozwojowym może być przeliczenie chmury punktów na podstawie dysparcji oraz następnie wykrywanie i omijanie przeszkód.

Poza samą realizacją stereowizji ważnym elementem był także optymalny przesył obrazu z obu kamer. Pierwszym podejściem było użycie MJPGstreamera. Efekty jednak nie były zadowalające. Klatki, które nie zostały jeszcze wysłane, były składowane w buforze. W wyniku czego, im dłużej działała aplikacja, tym większe było opóźnienie. Dodatkowo występowała różnica czasu pomiędzy klatkami z obu kamer, co w przypadku stereowizji jest nieakceptowalne. Dlatego zdecydowano się użyć gstreamera. Jest on bardziej zaawansowany, przez co uruchomienie go zajęło więcej czasu. Jednak uzyskany efekt jest bardzo dobry. W jego działaniu, jeśli nie zdąży się wysłać klatki przed przybyciem następnej, to zostaje ona porzucona. W wyniku tego wyeliminowane zostało rosnące opóźnienie. Dlatego też nie występowało już opóźnienie pomiędzy klatkami. Rozdzielczość przesyłanych obrazów wynosi 320x240. Przeprowadzono także testy dla rozdzielczości 640x480, ale spadek FPS był nieproporcjonalny do wzrostu jakości.

{% include button.html text="Stereowizja" icon="github" link="https://github.com/macstepien/HexapodStereovision" color="#0366d6" %}

### Osiągnięcia

#### Zawody robotyczne

- Robocomp 2017 w Krakowie (kategoria Freestyle)
- Robotic Arena 2017 we Wrocławiu (kategorie Freestyle oraz wyścig robotów kroczących). Pierwsze miejsce w kategorii wyścig robotów kroczących.
- Robomaticon 2018 w Warszawie (kategoria Freestyle)
- Robotic Tournament 2018 w Rybniku (kategoria Freestyle)
- Robotic Arena 2019 we Wrocławiu. Drugie miejsce w kategorii wyścig robotów kroczących.

#### Wydarzenia

- Targi pracy Kariera IT
- TEDxAGHUniversity

{% include figure.html image="/pics/11_hexapod/robotic_arena.jpg" width="400" height="800" caption="Prezentacja na zawodach Robotic Arena fot. Politechnika Wrocławska" %}

[Strona projektu](http://www.integra.agh.edu.pl/robot-kroczacy-freestyle/ "Strona projektu")
