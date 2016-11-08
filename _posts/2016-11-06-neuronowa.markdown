---
layout: post
mathjax: true
title:  "Prosta sieć neuronowa"
date:   2016-11-06
categories: jekyll update
---

Zaczęliśmy od metody ekstremalnie prostej w zrozumieniu, teraz będziemy piąć się trochę wyżej w skali nieprzystepności. Skorzystamy mianowicie z przeżywających ostatnio drugą młodość sieci neuronowych - zwanych tu i ówdzie wielowarstwowym perceptronem ( po wyjaśnienie, czym jest zwyczajny perceptron, zapraszam na Wikipedię, jest on jednakże archaiczną metodą uczenia statystycznego, więc nie sądzę, aby mógł się przydać. ).

Nie od razu sięgniemy po najświeższe zdobycze techniki, to znaczy sieci głębokie rekurencyjne i splotowe. Na pierwszy ogień pójdzie prosta sieć z jedną ukrytą warstwą, zaimplementowaną w googlowskiej zabawce TensorFlow. Nie możemy skorzystać z danych, które przetworzyliśmy na użytek łańcuchów Markova - dysponowaliśmy tam tylko informacją, czy wartość spółki wzrostła, czy spadła od punktu, w którym sprawdzaliśmy to po raz ostatni. Tym razem nie chcemy tracić potencjalnie pożytecznych informacji - w końcu wzrost o 10% to nie to samo, co marne kicnięcie kursu o ułamek procenta. W poprzednim modelu oba zostałyby zakwalifikowany tak samo, ze względu na potrzebę dyskretyzacji. Mała dygresja - można co prawda było wprowadzić trzecią wartość - ustawić próg kwalifikacji jako 'wzrost' i 'spadek' jako jakąś istotną bezwględną wartość, a całą resztą zapisywać jako 'w przybliżeniu stałe'. Ale $3^5 = 243$ to dużo więcej niż $2^5 = 32$, co więcej, dla większej części z możliwości mielibyśmy zapewne po prostu 0 wystąpień. 

Sedno klasy wydobywającej dane z folderu z ruchami cen jest podobne jak w łańcucach - mając ustalone przesunięcie $S$, przesuwamy się po kolei w kolumnie z cenami zamknięcie o $S$ i wydobywamy kolejne ceny. Tworzymy w ten sposób znaczną ilość wektorów o ustalonej zawczasu długości $L$. Będą to nasze wektory wejścia, które posłużą do uczenia i testowania sieci. Kod źródłowy znajduje się [tutaj](https://github.com/wig-ml/blog_source/blob/master/simple%20neural%20network/input_manager.py) 

Kontrowersyjną decyzją może być metoda ```rel``` - wszak różnica w cenie między obecnym położeniem czasowym $T$ a $T-i$ może nie być tak wartościową własnością. Jest to jednak sposób na znormalizowanie różnic w cenie, albowiem nie wydaje mi się, by spółka warta sto złotych miała mieć dużo lepsze ( bądź gorsze ) perspektywy na wzrost, niż jej konkurentka o wartości jednego złotego. Tymczasem, gdybym jako własność zachował nieprzetworzoną własność, tego typu różnice odgrywałyby dużą rolę - nawet ( a może w szczególności ) po przeskalowaniu cen tak, by ich średnia była równa 0, zaś odchylenie standardowe 1, czyli klasyczna obecna uczeniu maszynowym normalizacja.

Kolejnym problemem, jaki miałem nadzieję zacząć rozwiązywać poprzez przetworzenie danych w ten sposób było wprowadzenie do sieci relacji między kolejnymi cenami, z czym klasyczne sieci mają problemy - choć wprowadzanie dodatkowych warstw zbliża je do pozbycia się ich. W swoim doskonałym kursie sieci neuronowych (dostępny na [Courserze](https://www.coursera.org/learn/neural-networks/home)) Geoffrey Hinton zwraca uwagę na motywację wprowadzenia sieci splotowych. Otóż wyobraźmy sobie, że planujemy użyć zwykłej sieci neuronowej do rozpoznawania obrazów. Wyobraźmy sobie też, że ktoś wszystkie nasze obrazki treningowe pomieszał, ale wszystkie w jednakowy sposób. Co znaczy pomieszał?

Przypomnę, że obrazy przechowuje się w formie macierzy liczb. Zwyczajnych, dwuwymiarowych dla czarno-białych treści, dla kolorowych zaś mamy trzy macierze dwuwymiarowe, tych samych rozmiarów, nałożone na siebie. Każda z nich reprezentuje intensywność jednego z kolorów: czerwony, zielony i niebieski. Czasem dodaje się czwartą macierz - jeśli potrzebujemy wskazać nieprzezroczystości. A zatem, poprzez pomieszanie rozumiemy sprawienie, że dla każdego obrazu pixel $a_{ij}$ będzie teraz pixelem $a_{kl}$ i zachodzi przynajmniej jedno z równań $i \neq k$, $j \neq l$.

<div class="imgcap" style="margin:auto">
<img src='/assets/neural-net/permut.png'>
</div>

```python
z = np.ravel(X) #X to macierz obrazka z jedynką
np.reshape(z[np.random.permutation(z.shape[0])], X.shape) #macierz obrazka po prawej
```

Zwykła sieć neuronowa stwierdzi, że te obrazki są identyczne - czemu? Przypomnę może, jak wygląda taka sieć z jedną warstwą ukrytą - jeśli Czytelnik jest tego świadom, następny akapit może śmiało pominąć. 

<div class="imgcap">
<img src='/assets/neural-net/net-scheme.png'>
</div>

W praktyce sieć neuronowa to zestaw macierzy, a cała groźnie brzmiąca operacja "uczenia" to po prostu ich wymyślne mnożenie w jedną i drugą stronę. W najprostszym przypadku mamy warstwę wejść, ukrytą i wyjść. Każda jednostka jednostka $h_i$ jest połączona z warstwą wejść zestawem wag $\{ w_{ij} \}$, co sprawia, że przejście z jednej warstwy do drugiej to po prostu pomnożenie wejściowego wektora przez macierz wag. Jeśli długość wektora to $N$, a wielkość warstwy ukrytej $H$, to wielkość macierzy wynosi $N \times H$ - zakładając, że wektor jest kolumnowy. Warstwę ukrytą okraszamy nieliniową funkcją aktywacji, stosując ją do każdego elementu - zgodnie z [twierdzeniem o aproksymacji uniwersalnej](https://en.wikipedia.org/wiki/Universal_approximation_theorem). Spośród najczęściej stosowanych mamy funkcję logistyczną $\frac{1}{1+exp(-x)}$, ReLU $max(0,x)$ czy tangens hiperboliczny $\frac{exp(x)-exp(-x)}{exp(x)+exp(-x)}$. Omówienie różnic można znaleźć na przykład [tutaj](https://www.quora.com/What-is-special-about-rectifier-neural-units-used-in-NN-learning), ja wspomnę tylko, że obecnie prym wiedzie ReLU. Dla następnej warstwy traktujemy ukrytą warstwę jako wektor i w analogiczny sposób tworzymy macierz wag.

Dla problemu klasyfikacji na warstwie wyjść zazwyczaj stosuje się funkcję zwaną softmaxem. Jeśli $\mathbb{W}$ jest wektorem wyjściowym, to dla każdego jego elementu $W_i$ stosujemy $\frac{exp(W_i)}{\sum exp(W_j)}$. W ten sposób mamy dystrybucję prawdopodobieństwa wystąpienia każdej z klasy wyjściowych. Aby nauczyć naszą sieć czegoś użytecznego, należy teraz wykonać "propagację wsteczną" - w praktyce jest to po prostu implementacja reguły łańcuchowej dla obliczania pochodnych. Nie sądzę, żebym dał radę wytłumaczyć to lepiej niż [Micheal Nielsen](http://neuralnetworksanddeeplearning.com/chap2.html)

Odarta z wszelkich fajerwerków implementacja w TensorFlow wygląda tak:

```python
learning_rate=1e-3
batch=100
display=int(1e2)
layer_size = 256
iterations = int(4e3)

input_size = X.shape[1]
output_size = Y.shape[1]

x = tf.placeholder('float', [None, input_size])
y = tf.placeholder('float', [None, output_size])

weights = {
    'layer1': tf.Variable(tf.random_normal([input_size, layer_size])),
    'out': tf.Variable(tf.random_normal([layer_size, output_size]))
    }

biases = {
    'layer1': tf.Variable(tf.random_normal([layer_size])),
    'out': tf.Variable(tf.random_normal([output_size]))
    }

def MLP(x, weigths, biases):
    hidden = tf.add(tf.matmul(x, weights['layer1']), biases['layer1'])
    hidden = tf.nn.relu(hidden)

    out = tf.add(tf.matmul(hidden, weights['out']), biases['out'])
    return out

out = MLP(x, weights, biases)
loss = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(out, y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(loss)

init = tf.initialize_all_variables()

def get_batch(x, y, batch, iteration):
    L = x.shape[0]
    start = (batch*iteration-batch)%L
    return x[start:start+batch], y[start:start+batch]

with tf.Session() as sess:
    sess.run(init)

    for i in range(iterations):
       data, labels = get_batch(X, Y, batch, i)

       _, c = sess.run([optimizer, loss], feed_dict={x:data, y:labels})


```

Na szczęście moduł ```optimizer``` pozbawia nas konieczności pisania propagacji wstecznej od zera. Nie byłby to problem w tym wypadku, natomiast w przypadku bardziej skomplikowanych i subtelnych konstrukcji czekałyby mnie pewno godziny debugowania, aby zlokalizować ten mały detal, który wszystko psuje :) Jeśli Czytelnik ma ochotę obejrzeć implementację od zera - za pomocą samego pythona wspomaganego numpy - to zapraszam [tutaj](http://cs231n.github.io/neural-networks-case-study/)

Przypomnijmy, że podajemy jej zwykły wektor, jednowymiarowy, nie dwuwymiarową macierz, na której widoczne jak na dłoni są relacje przestrzenne, krawędzie, gradienty i tak dalej. W szczególności, dwa punkty leżące obok siebie w macierzy, w reprezentacji wektorowej mogą być oddalone o kilkaset pozycji! Rozważmy teraz pierwszą jednostkę z warstwy ukrytej. 

Niech $x_{i}$ będzie $i$-tą pozycją z wektora wejściowego, a $w_{i}$ wagą, z jaką podaje się go do pierwszej jednostki, $a_1$. Mamy wszakże $a_1 = \sum w_i x_i $, czyli kombinację liniową, niepodatną na zmianę kolejności dodawania! Możemy zatem dowolnie permutować elementy wejściowego wektora, a dopóki dysponujemy odpowiednim narzędziem optymalizacyjnym, powinniśmy otrzymać jakiś sensowny rezultat, pod warunkiem spełnienia twierdzenia o uniwersalnej aproksymacji. Analogiczną rzecz można powiedzieć o każdej jednostce, rzecz jasna. 

Aby na własne oczy przekonać się o tym, jak pomocne bywa "wkładanie własności ręką", Czytelnik może zajrzeć na [plac zabaw](http://playground.tensorflow.org/). Najlepszym chyba przykładem będzie wybranie spirali i sprawdzenie, jak sprawuje się sieć z sinusoidalnymi własnościami i bez nich. Choć w tym wypadku łatwo jest dobrać prawidłowe i pomocne własności, bo możemy sobie narysować zbiór danych, w ogólności nie jest to takie proste i należy uważać na [niedopasowanie i przeuczenie](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff)

Cały ten wywód nie zmienia faktu, że dla pojedynczej warstwy ukrytej rezultaty uczenia nie zachwycają. Otrzymujemy funkcję kosztu w granicach 0.70 - tyle wynosi uśredniona wartość dla każdego wektora ze zbioru testowego na końcu uczenia. Jeszcze raz z definicji entropii krzyżowej:
<div>
$$\vec{v_{przew}} = softmax( \vec{v_{przew}} )$$
$$C( \vec{v_{przew}}, \vec{v_{praw}} ) = - \sum \vec{v_{praw}}(i) \log \vec{v_{przew}}(i) = - \log \vec{v_{przew}}(j) $$
</div>

gdzie $j$ jest prawdziwą klasą - zauważmy, że dla całej reszty $\vec{v_{praw}}(i) = 0$. W podstawie logarytmu rezyduje liczba Eulera $e$, a zatem

<div>
$$\log x = -0.693 \implies x = 1/2$$
</div>

Własnie tak, nasza sieć nie sprawdza się dużo lepiej niż strzelanie. Kilka wykresów:

<div class="imgcap">
<img src="/assets/neural-net/firststats.png">
</div>

Widać, że funkcja kosztu bardzo szybko ląduje w swoich najmniejszych wartościach. Winowajcą słabej sytuacji raczej nie jest więc przeuczenie - choć dodałem dodatkową warstwę ukrytą. Przed uczeniem zmieniłem też inicjalizację zmiennych na 

```
tf.Variable(tf.random_normal([input_size, layer1], stddev=1/np.sqrt(input_size)))
```

Chciałem w ten sposób zapobiec eksplozji wag i utrzymać je w pobliżu zera - choć przy funkcji aktywacji ReLU zamiast sigmoidalnych jest to mniej prawdopodobne. Ta pierwsza ma bowiem pochodną równą jeden dla każdego dodatniego wejścia, zaś gradient drugiej spada do zera dla dużych co do modułu argumentów. Jest to niebezpieczne, ponieważ w tej sytuacji ciężko byłoby wydostać się z obszarów, gdzie wartość funkcji kosztu jest płaska. Prawdę mówiąc, podejrzewam, że winowajcą może być brak odpowiednich własności w sieci, ewentualnie za duży szum. Co się zatem stanie, gdy ograniczę zbiór treningowy do okresu po bańce z 2008 roku? Przy dokładnie takich samych parametrach sieci, niestety nie widać, żeby za dużo się zmieniło.

Wydaje mi się, że mogę podawać sieci za mało informacji - choć z drugiej strony, przecież kilkakrotnie przechodzi ona przez zbiór treningowy, ponadto bardzo długo oscyluje wokół jednej wartości funkcji kosztu, a zmiana tempa uczenia na rząd wielkości wyższe też nie pomaga. Ponadto sieć pokazana przez Karpathy'ego w CS231 sprawnie uczy się spiralnego zbioru danych. No, ale w akcie desperacji - spróbuję.

Mam wrażenie, że jest pewna poprawa - jeśli spojrzeć na dokładność sieci, to okazuje się, że na zbiorze treningowym zdarza się jej zejść poniżej 0.6. Niestety nie dotyczy to testowego, a ponadto ani precyzja, ani czułość nie pozwalają na nadmierny entuzjazm. Tym smutnym wnioskiem zakończę więc rozważania na temat zwykłych sieci neuronowych.

<div class="imgcap">
<img src='/assets/neural-net/typowy.png'>
</div>
Typowy wykres dla losowo wygenerowanych parametrów $S, L$ i tempa uczenia. 


