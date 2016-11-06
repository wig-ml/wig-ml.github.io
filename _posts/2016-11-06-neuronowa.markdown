---
layout: post
mathjax: true
title:  "Prosta sieć neuronowa"
date:   2016-11-06
categories: jekyll update
---

Zaczęliśmy od metody ekstremalnie prostej w zrozumieniu, teraz będziemy piąć się trochę wyżej w skali nieprzystepności. Skorzystamy mianowicie z przeżywających ostatnio drugą młodość sieci neuronowych - zwanych tu i ówdzie wielowarstwowym perceptronem ( po wyjaśnienie, czym jest zwyczajny perceptron, zapraszam na Wikipedię, jest on jednakże archaiczną metodą uczenia statystycznego, więc nie sądzę, aby mógł się przydać. ). 

Nie od razu sięgniemy po najświeższe zdobycze techniki, to znaczy sieci głębokie rekurencyjne i splotowe. Na pierwszy ogień pójdzie prosta sieć z jedną ukrytą warstwą, zaimplementowaną w googlowskiej zabawce TensorFlow. Nie możemy skorzystać z danych, które przetworzyliśmy na użytek łańcuchów Markova - dysponowaliśmy tam tylko informacją, czy wartość spółki wzrostła, czy spadła od punktu, w którym sprawdzaliśmy to po raz ostatni. Tym razem nie chcemy tracić potencjalnie pożytecznych informacji - w końcu wzrost o 10% to nie to samo, co marne kicnięcie kursu o ułamek procenta. W poprzednim modelu oba zostałyby zakwalifikowany tak samo, ze względu na potrzebę dyskretyzacji. Mała dygresja - można co prawda było wprowadzić trzecią wartość - ustawić próg kwalifikacji jako 'wzrost' i 'spadek' jako jakąś istotną bezwględną wartość, a całą resztą zapisywać jako 'w przybliżeniu stałe'. Ale $3^5 = 243$ to dużo więcej niż $2^5 = 32$, co więcej, dla większej części z możliwości mielibyśmy zapewne po prostu 0 wystąpień. 

Sedno klasy wydobywającej dane z folderu z ruchami cen jest podobne jak w łańcucach - mając ustalone przesunięcie $S$, przesuwamy się po kolei w kolumnie z cenami zamknięcie o $S$ i wydobywamy kolejne ceny. Tworzymy w ten sposób znaczną ilość wektorów o ustalonej zawczasu długości $L$. Będą to nasze wektory wejścia, które posłużą do uczenia i testowania sieci. Kod źródłowy znajduje się tutaj:


