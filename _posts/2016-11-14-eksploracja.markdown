---
layout: post
mathjax: true
title:  "Eksploracja danych"
date:   2016-11-14
categories: jekyll update
---


Nadszedł czas na analizę eksploracyjną - wezmę najpopularniejsze wskaźniki ruchów cen i sprawdzę, czy są w stanie służyć jako jakikolwiek predyktor ruchów cen akcji. Skrypty, z których korzystałem znajdują się [tutaj](https://github.com/wig-ml/blog_source/blob/master/data%20exploration/exp.py)

## SIŁA RELATYWNA

Czym jest siła relatywna? Niestety pod tą nazwą spotyka się dwa wskaźniki - przetestuję obydwa. 

Pierwszy to wskaźnik, który ma informać nas, jak się miała do siebie w danym okresie średnia wzrostów i spadków. Pozostając w przestrzeni "łańcuchów" różnego skoku i długości zdefiniowanych przy okazji n-gramów, niech $L$ będzie długością łańcucha, a $S$ skokiem. $C(t)$ będzie ceną w punkcie $t$ od początku. Wtedy $S_+ = \sum_{t \in A} ( C(t)-C(t-1) )/C(t-1) $, $S_- = \sum_{t \in B} ( C(t)-C(t-1) )/C(t-1)$, gdzie
<div style="margin:auto">
 $A = \{ t: C(t)-C(t-1) > 0 \}, B = \{ t: C(t)-C(t-1) < 0 \}$. 
</div>

Wtedy $RS = \frac{ S_+ / \mid S_+ \mid }{ S_- / \mid S_- \mid }$, a $RSI = 100(1 - \frac{1}{1+RS})$

Wyniki?

<div class="imgcap">
<img src="/assets/exp/RSI.png">
</div>

Funkcja ```transform_RSI```.

Może słowo wyjaśnienia, co obrazują powyższe wykresy. Tytuły przedstawiają parametry, które zostały wykorzystane do stworzenia wykresu - mamy oddalenia zarówno jednodniowe, siedmiodniowe, dwutygodniowe i miesięczne - zdecydowałem się przedstawić losową próbkę z 24 losowo wybranych zestawów. Redukując zmienne odstające - to jest zrównując pierwszy i ostatni percentyl z ich wewnętrznymi wartościami narysowałem wykres gęstości prawdopodobieństwa na podstawie histogramu z 50 koszyków. Nie jest to wykres prawdopodobieństwa wzrostu/spadku. Jest to wykres prawdopodobieństwa warunkowego, $P( X \in (a,b) \mid \Delta X > 0 )$ i $P( X \in (a,b) \mid \Delta X < 0 )$, czyli prawdopodobieńśtwa, że wartość spółki znajduje się w zadanym przedziale (jednym z pięćdziesięciu), jeśli jej wartość spadła/wzrosła. Zdecydowałem się na przedstawienie tego wykresu, ponieważ, obserwacji spadających było nieproporcjonalnie więcej niż rosnących - można to sprawdzić samodzielnie, ale niżej reprezentatywny dla całości obrazek.

<div class="imgcap">
<img src="/assets/exp/overall.png">
</div>

Funkcja ```transform_to_RS```

Na pierwszym zestawie wykresów widać niewielkie różnice między poszczególnymi dystrybucjami. W szczególności widzimy różnicę na korzyść wzrostu i spadku tam, gdzie sugerowałaby to interpretacja z [Investopedii](http://www.investopedia.com/terms/r/rsi.asp) - dla niższych wartości mamy wyprzedany papier, który czeka zwrot w górę, dla wyższych wykupiony, napompowany, który może niedługo pęknąć i spaść w dół.

Druga metoda polega na sprawdzeniu, jak wzrost każdej spółki ma się do najlepszej dzisiaj spółki. 

<div class="imgcap">
<img src="/assets/exp/ROC.png">
</div>

Nic imponującego - wręcz przeciwnie. Z drugiej strony, byłem przekonany, że metoda powinna przynieść satysfakcjonujące rezultaty. Intuicyjnie, jeśli spółka była lepsza od pozostałych przez ostatni miesiąc, to rzesza inwestorów powinna ten wzrost zauważyć i zainwestować w papier - tak właśnie powstają bańki. Czyżbym coś zepsuł? // tutaj link do kodu na github


## WSKAŹNIK SHARPE'A

W pewnym momencie historii rynków finansowych ktoś, a mianowicie facet nazwiskiem William Sharpe postanowił wprowadzić miarę rentowności papieru wartościowego z ryzyka, jakie podejmujemy inwestując weń. Ryzyko definiujemy tu jako odchylenie standardowe ruchów danego papieru. Wzór:
<div>
$$\frac{ R_p - R_b }{ \sigma_p }$$
</div>

gdzie $R_p$ to średni zwrot z inwestycji w papier w zadanym okresie, $R_b$ to średni zwrot z inwestycji w papier określony jako "bezpieczny" - tutaj niech będzie WIG - choć w WIG bezpośrednio inwestować nie można ( chyba, że skorzystamy z funduszu indeksowego ). $\sigma_p$ to własnie ryzyko inwestycji w nasz papier. Rozważę dwa przypadki: pierwszy, czyli średni zwrot tylko i wyłącznie w danym łańcuchu, do którego przypisany jest zwrot i drugi, czyli średnia ruchoma zwrotów z łańcuchów od początku.

<div class="imgcap">
<img src="/assets/exp/sharpes.png">
</div>

Funkcja ```transform_to_sharpe```

Powyżej wykres prawdopodobieństw warunkowych dla średnich zwrotów w danym łańcuchu - widzimy, że występuje między nimi pewna, choć nieznaczna różnica. Nasuwa się hipoteza, żeby sprawdzić, czy wskaźnik Sharpe'a razem z pierwszą odmianą RSI będzie wskazywał nam jakąś separowalność. Tymczasem sprawdźmy wersję dla średniej ważonej o kroku 0.8:

<div class="imgcap">
<img src="/assets/exp/sharpesemas.png">
</div>

Funkcja ```transform_to_sharpe_EMA```

Żeby nie zaśmiecać posta, zostawię tylko spostrzeżenie, że dla kroku 0.5 wygląda to podobnie. Wydaje się więc, że to podejście nie wróży nam pełnego skarbca - choć zastanawia mnie, skąd bierze się nagły spadek przy zerze. 

Warte uwagi są przynajmniej dwie rzeczy: przewaga rosnących akcji po lewej stronie krzywej, to jest dla ujemnych wskaźników. To sugerowałoby, że częściej można zarobić na odwracającym się trendzie, zamiast na jego kontynuacji - tak, jak w przypadku RSI. Po drugie, niepokoi mnie gwałtowny spadek wartości wskaźnika dla średnich ważonych - prawdę mówiąć, moją pierwszą myślą jest pomyłka w kodzie Pythona odpowiedzialny za obliczenia. Niemniej, warto sprawdzić, na ile wskaźnik Sharpe'a jest niezależny od siły relatywnej. Jeśli korelacja nie jest wysoka, będzie to istotna, dodatkowa informacja, którą warto się zasugerować. 


## ŚREDNIE WAŻONE

Ostatnią sekcją w tej części będzie analiza rezultatów inwestowania według średnich... różnych.

1. Średnia różnica cen ważona różnicą w wolumenie.
2. Średnia różnica cen ważona wolumenem.
3. Zwykła średnia cen w łańcuchu podzielona przez ostatnią cenę.
4. Średnia ruchoma wykładnicza z różnic cen.

<div class="imgcap">
<img src="/assets/exp/avgs.png">
</div>

Funkcja ```transform_average```

Kolejne wykresy średnich dla skoku łańcucha 2, długości 18 i ceny następnego dnia.


<div>
<img src="/assets/exp/avgs_.png">
</div>

Skok - 6, długość - 14, cena dwa tygodnie później.


Istnieje pewna zauważalna, choć niewielka, różnica między dystrybucjami czerwoną a niebieską. Zastanawiające jest, że mamy tutaj wyraźnie widoczną przewagę po stronie dodatnich wartości, choć dla wskaźnika Sharpe'a i tempa zmian wszystko, co dało się wypatrzeć to dramatyczne załamanie i dominujący czerwień błękit - potwierdza to tylko moje podejrzenie, że popełniłem gdzieś błąd. Nie udało mi się niestety go namierzyć. 

Nie zamieszczałem wszystkich wykresów - wszystkim i tak nikt by się nie przyjrzał, a dodatkowe można z łatwością wygenerować posiadając gotowe funkcje ze skryptu na początku postu.

Podsumowując, myślę, że istnieje nadzieja na wykorzystanie tych wskaźników jako własności ( zmiennych objaśniajacych ) ostatecznego modelu. Widać nieznaczną separowalność, a przy dobrych wiatrach może się okazać, że wskaźniki są ze sobą słabo skorelowane. Będziemy wtedy wiedzieć, że zyskujemy nową informację włączając wskaźnik do modelu, zamiast powielania już istniejącej. 

Przed nami jednak testy, dobranie najbardziej odpowiadających konkretnym wskaźnikom łańcuchów. W następnym odcinku postaram się skorzystać z twierdzenia Bayesa celem znalezienia strategii - podobnie, jak w odcinku o n-gramach. Poza tym spróbuję sprawdzić i włączyć do modelu więcej wskaźników, a następnie raz jeszcze przetrenuję sieć neuronową. Dalsze plany zakładają podzielenie ruchów cen akcji na trendy rosnące i malejące i zastosowanie metod analizy przeżycia do zidentyfikowania czasu pozostałego do odwrócenia trendu.
