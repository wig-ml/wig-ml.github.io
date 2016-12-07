---
layout: post
mathjax: true
title:  "Regresje"
date:   2016-12-2
categories: jekyll update
---

Nadszedł w końcu czas na połączenie kilku wskaźników ze sobą. Zacznę od zwykłej regresji logistycznej - na początku korzystając z pakietu wszystkich omówionych do tej pory wskaźników, później będę kombinować i łączyć je ze sobą na różne sposoby? Ale czym właściwie jest regresja logistyczna? Jeśli już wiesz, to możesz pominąć poniższy akapit.





<div>
<img src="/assets/regression/histonereach.png">
</div>

Dla przypomnienia, histogram wskaźników jednego z łańcuchów. Widzimy rozkład z grubsza normalny, co ułatwi nam stosowanie testów statystycznych, gdy będziemy później upraszczać bądź ubogacać model. Wierzchołki na boku to artefakt powstały z powodu dobranego sposobu koszykowania - skrajne wartości, to jest te powyżej 98. i poniżej 2. percentyla zostały umieszczone w jednym koszyku. Na początek można sprawdzić, jakie rezultaty osiągniemy jako wynik testu ANOVA zastosowanego do każdej z własności. 

Co to jest test ANOVA? Analysis of variance, czyli analiza wariancji. Służy do określania, na ile podział na grupy jakościowe determinuje różnice w zmiennych jakościowych. Innymi słowy, jeśli mamy $k$ grup, to chcemy dowiedzieć się, czy będą one się jakoś istotnie różnić od siebie. W szczególności pozwoli nam to stwierdzić, czy możemy losowo wybraną obserwację z losowo wybranej grupy przyporządkować do którejś grupy bez wiedzy o tym, skąd została ona pobrana. Z czym to się je?

//opis


Weźmy przykładowy łańcuch, na przykład S1L18R1. Jak każda zmienna ma się do wymaganych założeń? Do sprawdzenia normalności użyję testu [https://en.wikipedia.org/wiki/Shapiro%E2%80%93Wilk_test](Shapiro-Wilka), do sprawdzenia niezależności posłuży mi rzecz jasna kowariancja. 

A zatem:

|                | ANOVA | shapiro - 1 | shapiro - 0 | wariancja - 1 | wariancja - 0 | kowariancja |
|----------------|-------|-------------|-------------|---------------|---------------|-------------|
| RSI            | 0.0   | 0.24        | 0.03        | 0.94          | 1.04          | 1.04        |
| sharpe-treynor | 0.09  | 0.0         | 0.0         | 0.56          | 1.39          | 1.39        |
| avg-exp        | 0.0   | 0.0         | 0.0         | 1.81          | 0.52          | 0.52        |
| avg-simple     | 0.39  | 0.0         | 0.0         | 0.55          | 1.73          | 1.73        |
| avg-vol        | 0.34  | 0.0         | 0.0         | 0.68          | 1.46          | 1.46        |
| avg-rel-vol    | 0.0   | 0.0         | 0.0         | 0.63          | 1.61          | 1.61        |
| sharpe-sharpe  | 0.44  | 0.0         | 0.0         | 0.91          | 1.07          | 1.07        |


No cóż, nawet jeśli otrzymaliśmy w trzech wypadkach ekstremalnie niską wartość prawdopodobieństwa, że te dwie wartości są istotnie różne, to stało się tak prawdopodobnie z powodu naruszenia założeń - kowariancja jest mocno niezerowa, wariancja różni się kilkukrotnie, tylko test Shapiro wskazuje wymaganą normalność - i to nie zawsze. Wobec tego nie spodziewałbym się fenomenalnych wyników regresji, ale sprawdzić nie zaszkodzi. 

<div>
<img src="/assets/regression/loss.png">
</div>

Co można wywnioskować? Na szczęście udało się nie wpaść w przeuczenie, więc mamy jeden problem z głowy - nie zmienia to faktu, że potencjał przewidywania jest dość marny. Wystarczy sobie przypomnieć wzór na funkcję kosztu - widać, że zbiega ona do okolic $0.6 - 0.7$. Po spojrzeniu w wartości okazuje się, że jest to dokładnie 0.669 dla ostatniej iteracji. Średnia pewność wskazań to $\exp -0.669 = 0.511$. Cudów spodziewać się nie można. Przejdźmy do mięsa, czyli dokładności, czułości i precyzji. Tak jak poprzednio, można dobrać próg akceptacji wyniku jako 'wzrostowy' dowolnie - jest to wręcz konieczne do dalszej weryfikacji modelu.

<div>
<img src="/assets/regression/ROC.png">
</div>

Wygląda to nieco lepiej niż poprzednio (fakt, że teraz wykres jest większy, czyli nieco łatwiej dojrzeć rozbieżność między prostą a krzywą). Przypuszczenia potwierdza wartość pola pod krzywą - 0.55. Jak zaś wygląda dokładność, czułość i precyzja w funkcji progu?

<div>
<img src="/assets/regression/rpa.png">
</div>

Najlepiej będzie zatem inwestować dobierając próg 'wzrostu' w okolicach .40, tak jak poprzednio. Wyławiamy wtedy mniej więcej połowę ze wszystkich wartości prawdziwie wzrostowych, a nieco mniej niż połowa wszystkich wskazań wzrostowych jest naprawdę wzrostowa. W porównaniu do alternatyw nie wygląda to tak źle.

Skoro już o inwestowaniu mowa, jak to wygląda w praktyce? Różnie, w zależności od tego, ile najlepszych akcji dopuścimy do głosu - jest to drobna modyfikacja metody z poprzedniego odcinka, choć nieznaczna.

<div>
<img src="/assets/regression/progfirst.png">
</div>

K jest liczbą papierów, które bierzemy pod uwagę do kupna w jednym kroku, S - do sprzedaży. Próg uznania za akcję wzrostową to 0.40. Dla wyższych wartości otrzymuję 

Poza regresją logistyczną warto jeszcze sprawdzić regresję liniową. Niestety, w zalezności od przyjętego progu akceptacji, wskazuje taki sam kierunek, jak logistyczna w 40\% do 60\% przypadków. Z rzeczywistymi wskazaniami też nie ma zbyt wiele wspólnego - kierunek zgadza się raptem w 50\% przypadków. Ale! Może okaże się, że głównie myli się przy słabych przesunięciach. Wiedzielibyśmy wtedy, że tego typu wskazania są mało wiarygodne i możnaby wstrzymać się z decyzją inwestycyjną. Taką alternatywę sugeruje mi fakt, że pierwiastek średniej funkcji kosztu okazuje się być niski - w szczególności niższy niż 0.10.

<div>
<img src="/assets/regression/losslin.png">
</div>

<div>
<img src="/assets/regression/agreement.png">
</div>


```python

lin_pred = lin.predict(X[t])

correct = np.where((lin_pred>0)==Y[t])
incorrect = np.where((lin_pred>0)!=Y[t])

print(np.sqrt(lin.calculate_cost(X[t][correct], roc.price_movs[t][correct])))

print(np.sqrt(lin.calculate_cost(X[t][incorrect], roc.price_movs[t][incorrect])))

```

Powyższy kawałek kodu objawia nam, że w przypadku prawidłowego przewidywania kierunku funkcja kosztu jest wyższa niźli dla przewidywań niepoprawnych, odpowiednio 0.05 i 0.04. Gdyby moja hipoteza była prawdziwa, mniej więcej czegoś takiego bym się spodziewał - dla wyższych ruchów cen mamy więcej miejsca na pomyłkę.

<div>
<img src="/assets/regression/corrincorr.png">
</div>

Niestety wykres gęstości prawdopodobieństwa dla przewidywanych ruchów akcji raczej obala tę teorię - gdyby była prawdziwa, widać by było większą gęstość krzywej niebieskiej w okolicy zera, a zielonej w większym oddaleniu, na kształt dwóch wzgórz z doliną w okolicy zera. Tymczasem są one podobne, tylko nieco przesunięte na osi X. Nasuwa się refleksja, która powinna być oczywista od początku - dane dotyczą przesunięcia jednodniowego (dla tego łańcucha), nie zaś miesięcznego czy rocznego. Okazuje się, że średnia wartość bezwzględna przesunięcia to 0.02. W takiej sytuacji pozornie mała funkcja kosztu przestaje dziwić. Gdyby była rząd wielkości mniejsza - to jest, gdybyśmy mówili o rzędzie wielkości tysięcznych, wtedy takie podejrzenia miałyby rację bytu.

<div>
<img src="/assets/regression/lineinvest.png">
</div>


Powyżej wyniki z inwestowania podług regresji liniowej. Kupno spółki, jeśli przewidywania są większe niż 0.05, sprzedaż - gdy mniejsze niźli -0.05. Po roku portfel niestety się zmniejsza, zadziwiające jest podobieństwo podejścia sprawdzającego największą i najmniejszą część akcji.

<div>
<img src="/assets/regression/linthreshold0.png">
</div>

Jeśli do inwestowania wystarczy nam, że model przewiduje wzrost, to stracimy mnóstwo pieniędzy.

<div>
<img src="/assets/regression/losslinabs.png">
</div>

Powyżej wykres funkcji kosztu dla trenowania regresji liniowej wartościami bezwzględnymi ruchów cen. Jak widać, funkcja kosztu znajduje się powyżej linii wskazującej średnią ruchów cen - zdecydowanie wskazuje to, że mają ten sam rząd wielkości. 


Na koniec warto jeszcze sprawdzić, czy wystąpią istotne różnice po ograniczeniu modelu. Wybrałem trzy zmienne, którym udało się osiągnąć zerową p-wartość ANOVy - wiedząc, że to prawdopodobnie nic nie znaczy. 


<div>
<img src="/assets/regression/truncinvest.png">
</div>

<div>
<img src="/assets/regression/truncrpa.png">
</div>

<div>
<img src="/assets/regression/ROCtrunc.png">
</div>

Nie widać specjalnej różnicy - inwestując w dalszym ciągu tracimy, krzywe precyzji, czułości i dokładności wyglądają podobnie, krzywa ROC również, zaś pole pod krzywą wynosi 0.54 - nieco mniej niż w pełnym modelu. 

Czas nadszedł w końcu na sprawdzenie różnych łańcuchów:

<div>
<img src="/assets/regression/gridofrocs.png">
</div>

AUC:
S1L15R7 0.540
S2L18R1 0.544
S7L11R30 0.518
S6L14R14 0.523
S2L6R30 0.512
S4L6R7 0.537
S4L17R30 0.515
S1L18R1 0.554
S3L6R14 0.538

Ostatnim modelem, który pójdzie pod lupę, będzie nawiązujący do pierwotnych pomysłów - sprawdzę bowiem, jak zachowują się portfele pod kontrolą regresji, której podajemy zmiany akcji względem poprzedniego punktu. 

<div>
<img src="/assets/regression/changesprogre.png">
</div>

Dla pierwszego z nich krzywe czułości, precyzji i dokładności wyglądają tak:

<div>
<img src="/assets/regression/rpachanges.png">
</div>

AUC:
S1L15R7 0.521
S2L18R1 0.513
S7L11R30 0.509
S6L14R14 0.507
S2L6R30 0.504
S4L6R7 0.528
S4L17R30 0.500
S1L18R1 0.510
S3L6R14 0.517


Na tym chyba można zamknąć temat - regresja liniowa i logistyczna okazały się być nie bardziej skuteczne niż metody bardziej naiwne. W odwodzie pozostają jeszcze metody drzewowe - drzewa decyzyjne, lasy losowe. Ponadto sięgnę jeszcze po analizę przeżycia celem dowiedzenia się, kiedy można spodziewać się odwrócenia trendu. Mimo wszystko, w pewnym momencie trzeba będzie sięgnąć po więcej danych, które z wysokim prawdopodobieństwem trzeba będzie wyekstrahować z nieustrukturyzowanych newsów. 







