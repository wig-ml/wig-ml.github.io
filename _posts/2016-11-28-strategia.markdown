---
layout: post
mathjax: true
title:  "Strategie"
date:   2016-11-28
categories: jekyll update
---

Po bardzo wstępnej analizie eksploracyjnej pora na zabranie się za metrykę najwazniejszą - to jest, pieniądze. Na początek rozważę strategię naiwną, to jest zmierzę, jakie są średnie zyski z kupowania i sprzedawania, gdy strategia nam to podpowiada. Pominę rzecz jasna koszty transakcyjne, pozwalając sobie na wzięcie ich pod uwagę w późniejszym etapie. Do wyznaczania prawdopodobieństwa sprzedaży użyję stosunku obecnej ceny do średniej ostatnich cen. Co więc otrzymujemy?

<div>
<img src="/assets/money/pobrane.png">
</div>

Co tutaj widać? Każdy z tych czterech wykresów odpowiada jednemu łańcuchowi z zasięgiem równym 1. Na osi odciętych mamy kolejne progi kupna/sprzedaży. Na osi rzędnych siedzą średnie zyski dla danego progu. Czerwona krzywa to średni zysk z kupowania akcji, gdy strategia nam to podpowiada. Zielona to średni zysk ze sprzedaży, a konkretniej, zagrywek na 'krótką sprzedaż', czyli, w dużym skrócie, zyskujemy wtedy, gdy cena spada. Wyniki są, cóż, marne. Widzimy, że nie dość, że średnie zyski z zachowania byczego w jednym tylko wypadku są wyższe niż część tysięczna, to jeszcze tracimy majątek próbując grać na w sposób czysto spekulacyjny. Dramat.

Idziemy dalej. Tym razem zmierzę zyski ze strategii, którą z dużym marginesem błędu można nazwać rzeczywistą. Otóż dla pewnej metody, załóżmy średniej ceny przez aktualną cenę wyznaczę pewną liczbę $N=30$ najlepiej sprawujących się progów kupna $T$ (tj. tych o najwyższym F-ratio). Zapamiętuję też, rzecz jasna dystrybucję prawdopodobieństwa dla danego wskaźnika. Jeśli oznaczymy $D$ jako uporządkowaną sekwencję dat, to postępuję następująco.

Rozpoczynam wędrówkę od punktu startowego, w tym wypadku początku roku 2015, pierwszego dnia handlu w styczniu. Zbieram wszystkie ceny spółek będących tego dnia w obrocie i wybieram spośród nich 5 o największym i najmniejszym prawdopodobieństwie wzrostu. Ci wstępni kandydaci muszą jeszcze przejść następny etap selekcji - sprawdzenie, czy prawdopodobieństwo wzrostu/spadku jest powyżej/poniżej wybranego progu $t$. Spółki kwalifikujące się lądują w dwóch słownikach, jednym dla kupna, drugim dla sprzedaży. Najpierw, rzecz jasna, dla zwolnienia środków, odbywa się sprzedaż. Do kasy trafia równowartość akcji w danym momencie pomniejszona o prowizję maklerską. Następnie dla każdej spółki, której akcje przeszły selekcję, sprawdzamy, ile sztuk możemy zakupić. Posiadaną gotówkę mnożymy przez $(1-f)$ gdzie f to prowizja maklerska, a następnie dzielimy przez cenę akcji. Biorąc podłogę (największą liczbę całkowitą mniejszą od tej wartości), możemy się dowiedzieć, ile akcji w ten sposób jesteśmy w stanie zakupić. Następnie dokonujemy transakcji - aby sobie nieco uprościć życie, przyjąłem, że jeśli nie jesteśmy w stanie pokryć prowizji maklerskiej, to zaciągamy mikropożyczkę. Po wszystkim przechodzę do następnego punktu czasowego - jeśli mój łańcuch miał zasięg $R=1$, to jest to następny dzień. Jeśli $R=7$, to tydzień. Jak zatem radzę sobie z mnóstwem pominiętych akcji? Otóż, gdy dotrę już do końca cen akcji, zaczynam od początku, ale z przesunięciem o jeden dzień do przodu. 

Pełny algorytm dla pojedynczego łańcucha w pseudokodzie wygląda tak:

1. wytrenuj łańcuch L o zasięgu $R$
2. $D=0$
3. $\forall t \in T$
   * $\forall r \in \{0, ..., R-1\}$
      * $\forall d \in \{r, r+R, r+2R, ... \} $
      * wybierz 5 najlepszych i 5 najgorszych akcji
      * z najlepszych weź te większe od t, z najgorszych te mniejsze
      * dla $i$-tej pozytywnej kup $\underline{p_i \cdot (P-f)/C}$, gdzie $p_i$ to prawd. zwyżki, P posiadane pieniądze, C cena akcji, a f prowizja maklerska
      * sprzedaj wszystkie akcje nominowane do sprzedaży, jeśli posiadasz je w portfelu


Co krok ewaluujemy nasz portfel, mnożąc ilość posiadanych akcji przez obecną cenę. Może się okazać, że spółka wypadła z rynku, ale jest też możliwe, że z jakichś przyczyn jej nieobecność trwała nie dłużej niż jeden dzień. Wtedy sprawdzamy, jak daleko wstecz siegają jej ostatnie ślady bytności na rynku. Jeśli miało to miejsce ponad miesiąc temu, to usuwamy ją z portfela - pieniądze przepadają. Jeśli ostatnie znaki życia zanotowano później, liczymy wartość dla ostatniej zanotowanej ceny. Poprzednie wyniki, to jest z podejścia naiwnego na początku postu nie napawały optymizmem, ale zobaczmy, jak by wyglądała sytuacja, gdyby strategia była spójna.

<div>
<img src="/assets/money/progressions.png">
</div>
Na osi odciętych kolejne dni transakcyjne. 

Pierwszy kwadrat wykresów średnie wartości portfeli dla tych samych czterech łańcuchów, co wyżej, z takim samym sposobem szacowania prawdopodobieństwa wzrostu. Nie dla wszystkich progów wykonywane były jakiekolwiek transakcje, wziąłem tylko te, dla których otrzymywaliśmy jakiekolwiek sygnały - w praktyce oznaczało to pozbycie się wskazań dla kilku progów.  Ponadto, okazało się że po odrzuceniu nieaktywnych progów, wartości portfela dla aktywnych były dla siebie bardzo zbliżone ( jeśli nie takie same ). 

Tyle technicznych uwag. Widzimy, że zyski i straty były różne w zależności od tego, jakim łańcuchem się posługiwaliśmy - fatalnie sprawił się ten sięgający aż 77 dni transakcyjny wstecz, wygaszający drobne ruchy akcji, sprawdzając zmienność sześciodniową aż 11 razy. Gdybyśmy grali według jego wskazań, nasze portfel odnotowałby skandaliczny spadek sięgający nawet połowy wartości. Pozostałe dwa, sprawdzające zmienność dłuższą niż jeden dzień, też nie przyniosłyby nam fortuny, choć nadal wychodzimy korzystniej niż podążając za głównym indeksem. Fenomenem jest łańcuch zmienności jednodniowej, sięgający 18 dni transkacyjnych wstecz - zysku rzędu 50\% nie widuje się zbyt często. Może to być związane z faktem, że nasz horyzont czasowy jest krótki - prawdopodobieństwo wzrostu konstruowane było patrząc raptem jeden dzień do przodu, co skombinowane ze informacją o zmienności jednodniowej i stosunkowo małą długością łańcucha dało nam perfekcyjny wskaźnik. Gdyby nie wrodzony sceptycyzm, nasunęłaby mi się teoria, że im większa długość łańcucha, tym mniejsza korelacja z przewidywaniem jednodniowej zmienności. Intuicja podpowiadałaby, że może to być prawda, ale wyciaganie wniosków z czterech sampli jest zwyczajnie nieprzyzwoite i nie przystoi nikomu, kto chciałby chlubić się posiadaniem choć krzytyny zdrowego rozsądku. 

Inna sprawa, że tak wysoki zysk wydaje mi się zwyczajnie podejrzany i trzeba będzie sprawdzić, czy nie mam jakiegoś krzaka w kodzie - po pierwszym przejrzeniu nie mogę nic znaleźć, ale nie rozstrzyga to jeszcze definitywnie jego braku. Tym dziwniejsze, że przecież separowalność pokazana wcześniej była niewielka, a AUC 0.53 nie napawa optymizmem. Najrozsądniejsza hipoteza wyjaśniająca to zapewne dobry moment startu - na poparcie tezy wykres niżej.

<div>
<img src="/assets/money/nrs.png">
</div>

Powyżej znajduje się wykres uśrednionej ilości akcji w portfelu - dla czerwonego i niebieskiego najmniejszą wartością nie jest 0, tylko 4!!!. 

Czemu uważam, że ten wykres wspiera tezę o dobrym starcie? Zauważmy, że kupno akcji odbywa się tylko na początku z drobnym wyjątkiem sprzedaży i kupna trzy miesiace później. Fakt, wybraliśmy akcje, które nie kwalifikowały się na sprzedaż później - w ogólności jest to cecha każdego łańcucha. Dobrze to może świadczy o sposobie wyboru, ale pamiętajmy, że nasz model probabilistyczny nie zajmował się przewidywaniem, czy papier ma szansę na wzrost do końca roku, tylko czy ma szansę na wzrost następnego dnia!. Przydałoby się sprawdzić, jak często akcje były dokupywane po sprzyjającej selekcji, a jak często gromadzono je od nowa.

<div>
<img src="/assets/money/rebuys.png">
</div>

Pomarańczowy stosunkowo często sugerowałby dokupienie akcji - co jednak z tego, skoro tak samo fatalny postępował łańcuch niebieski. Zielonemu zdarzało się nawet sugerować poprawność wyboru dwóch spośród posiadanych w portfelu akcji. Łańcuch czerwony zaś był zupełnie niepewny swojego wyboru - dopiero pod koniec widzimy parę stalagmitów. Następnym krokiem będzie sprawdzenie, jak zachowa się wartośc portfela, jeśli będziemy cyklicznie go wyprzedawać i kupować akcje od nowa. Potwierdzenie dobrego wyboru otrzymujemy tutaj, bądź co bądź, dosyć rzadko, a nawet jeśli, to dotyczy ono jednej tylko spółki. Ponadto zauważmy, że dramatyczny spadek na niebieskim wykresie zbiega się z nagłym zmniejszeniem liczby akcji w portfelu - wtedy właśnie okazało się, że spółka, która wpadła do portfela zakończyła swoją bytność na rynku kapitałowym. Upłynnienie papierów zabezpieczyłoby nas przed takimi przygodami - ale z drugiej strony zwiększyłoby opłaty transakcyjne.

<div>
<img src="/assets/money/sprzedaz.png">
</div>

Zdjąłem warunek o zaciąganiu długu - teraz, jeśli transakcja miałaby nas zostawić z torbami, to zwyczajnie jej nie wykonujemy. Okazuje się, że jedyną odnotowywalną zmianą jest drobna modyfikacja kształtu wykresu wielkości portfela. Wszystko inne pozostaje bez zmian... co nieco dziwi, wszak opłaty transakcyjne powinny swoje zjeść - mając 10 różnych papierów, jednorazowo wydamy mniej więcej 500 złotych. Po wstawieniu tu i ówdzie poleceń wypisywania komunikatów okazuje się, że istotnie, nakazane sprzedaże algorytmowi się zwyczajnie nie opłacają, więc ich nie wykonuje, by nie generować długu.

To wszystko dotyczyło wskaźnika ceny do średniej. Co się stanie, jeśli zaprzęgniemy do roboty wskaźnik z drugim najwyższem wynikiem w tabeli AUC?

<div>
<img src="/assets/money/treynor.png">
</div>

Cóż, role się odwróciły - w trzech z czterech przypadków. Łańcuch, który poprzednio zdominował grę, teraz wyjmuje nam z portfela cztery tysiące złotych w ciągu roku, zaś dwa pozostałe generują zyski. Niebieski, o najdłuższym okresie patrzenia w tył, w dalszym ciągu jest słabym kompanem do gry.

<div>
<img src="/assets/money/treynornr.png">
</div>

Widać też, że w dalszym ciągu portfel się stabilizuje na samym początku. 

<div>
<img src="/assets/money/RSI.png">
</div>

Powyżej wykresy dla tych samych łańcuchów, ale z wykorzystaniem wskaźnika RSI. Tym razem aż dwa z nich okazują się przynosić wybitne zyski, choć znowu jest to kwestia dobrze dobranej transakcji na samym początku.

<div>
<img src="/assets/money/RSInr.png">
</div>

<div>
<img src="/assets/money/RSIrebuy.png">
</div>


Wobec faktu zróżnicowanej skuteczności wskaźników, może warto je połączyć? W następnym odcinku regresja logistyczna i (prawdopodobnie) AdaBoost.



