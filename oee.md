# OEE

## Calcolo KPI riferiti ad una risorsa

Qui di seguito le formule per calcolare gli indicatori **Availability**, **Perfomance** e **Qualità** in un turno per una determinata risorsa. Le formule sono comunque generalizzabili per intervalli di tempo diversi ( ad esempio l'ora ). E' solito usare il turno come campione per analisi storiche, mentre unità più piccole solo per analisi puntali come il turno in corso.

### Availability
L'indicatore **Availability** è determinato dal rapporto della disponibilità *effettiva* di una risorsa a produrre rispetto al tempo *previsto*
Abbiamo quindi a numeratore la durata del turno ridotta di tutti i fermi ( ossia tutto il tempo in cui la macchina era disponibile ), mentre a denominatore la durata del turno ridotta dei fermi pianificati

$$
A = \frac{ T_{sched} - T_{unsched}}{T_{sched} }  = \frac{ (S_e - S_s ) - \sum_{i} (P_{ie} - P_{is} ) - \sum_{j}  (U_{je} - U_{js} )}{(S_e - S_s ) -\sum_{i} (P_{ie} - P_{is} )}
$$


- $(S_f - S_s )$ è la durata del turno, espresso come differenza tra fine turno ( oppure adesso se turno in corso ) e inizio turno

- $\sum_{i} (P_{ie} - P_{is})$ è la durata del tempo di fermo pianificato, espresso come somma delle differenze della fine del fermo i-esimo ( oppure adesso se il fermo è in corso) e inizio del fermo

- $\sum_{j}  (U_{je} - U_{js} )$ è la durata del tempo di fermo *non* pianificato, espresso come somma delle differenze della fine del fermo j-esimo ( oppure adesso se fermo è in corso) e inizio del fermo

> E' solito non includere tra i fermi non programmati gli intervalli di indisponibilità la cui durata è inferiore ad una certa soglia ( i.e. *microfermate* ). Il loro *impatto" ricadrebbe nell'indicatore delle performance.

### Performance
L'indicatore **Performance** è determinato dal rapporto tra la *effettiva* velocità della macchina rispetto al tempo *effettivo* di produzione.
Abbiamo quindi a numeratore la somma dei pezzi prodotti nel turno per il tempo ciclo teorico a produrli, mentre a denominatore quello che è stato espresso a numeratore dell'availability.

$$
P = \frac{ T_{prod} }{T_{sched} - T_{unsched}} = \frac{\sum_{l}C_l * T_l + \sum_{k}C_k * T_k }{(S_e - S_s ) - \sum_{i} (P_{ie} - P_{is}) - \sum_{j}  (U_{je} - U_{js} )}
$$

- $\sum_{l}C_l * T_l$  è la somma  delle quantità di tutte le conferme di pezzi conformi  l  per il tempo ciclo teorico 

- $\sum_{k}C_k * T_k$  è la somma  delle quantità di tutte le conferme di pezzi non conformi k per il tempo ciclo teorico 

> E' possibile definire a livello di risorsa una tolleranza ( ad esempio 85% ) da applicare al tempo ciclo teorico.

 

## Qualità
L'indicatore **Qualità** è determinato dal rapporto tra la produzione di prodotto conforme e la produzione totale.
Abbiamo quindi a numeratore la somma dei pezzi conformi e a denominatore la somma totale dei pezzi prodotti

$$
Q = \frac{ Q_{buoni} }{ Q_{buoni} + Q_{scarti} } = \frac{\sum_{l}C_l}{\sum_{l}C_l + \sum_{k}C_k  } 
$$


## Calcolo KPI riferiti ad un gruppo di risorse

Il valore dei KPI riferito ad un gruppo di risorse può essere definito come la media dei KPI delle singole risorse solamente se tutte le macchine che compongono il gruppo sono schedulate per lavorare la stessa quantità di tempo

Una più sicura generalizzazione consiste nel considerare la somma totale dei tempi produttivi e quindi dei fermi e dei pezzi prodotti all'interno di questi intervalli.

> Un esempio numerico, ( anche se irrealistico ): supponendo di avere una risorsa pianificata per 8 ore in un giorno lavora con A=95% e un'altra pianificata per 2 ore lavora con A=60%, con la media semplice risulterebbe un $A_r$ di 77.5%. Considerando la somma dei tempi produttivi invece, avremmo un calcolo corretto del 88%

### Availability di reparto

La differenza con la **Availability** di una singola risorsa consiste nel considerare la somma di tutti gli intervalli produttivi di tutte le risorse r, la somma di tutti i fermi schedulati I e la somma di tutti i fermi non schedulati J.
Questo KPI indica la affidabilità del reparto.


$$
A_r = \frac{\sum_{r} (S_{rf} - S_{rs} ) - \sum_{I} (P_{Ie} - P_{Is} ) - \sum_{J}  (U_{Je} - U_{Js} )}{\sum_{r} (S_rf - S_rs ) -\sum_{I} (P_{Ie} - P_{Is} ) }
$$

#### Availability di una linea seriale.

Nel caso di una linea in cui le macchine lavorano in serie, è probabile che se una macchina della serie si fermi anche le macchine a monte si fermino o a valle si fermino ( magari non immediatamente perchè sono disponibili dei buffer. ) In questi casi, usando la formula precendente si avrebbero dei conteggi sbagliati. Un approccio più realistico potrebbe essere quello di considerare come *Availability* di questo gruppo di macchine come la minore availability registrata da una di queste ( che agisce da collo di bottiglia )

$$
A_s = \min(A_r)
$$

### Performance di reparto

Similarmente, per le performance si prendono in considerazione tutti i pezzi prodotti buoni L e scarto K  da tutte le risorse facente parte del reparto. Questo KPI indica la differenza tra la capacità teorica del reparto e quella effettiva.

$$
P_r = \frac{\sum_{L}C_L * T_L + \sum_{K}C_K * T_K }{\sum_{r} (S_rf - S_rs ) - \sum_{I} (P_{Ie} - P_{Is} ) - \sum_{J}  (U_{Je} - U_{Js} )}
$$

### Qualità di reparto

Si applica lo stesso ragionamento, si considerano i pezzi prodotti da tutte le macchine del reparto. Questo KPI può essere inteso come la probabilità che dato un pezzo eseguito in una macchina facente parte del gruppo, questo venga rendicontato come buono.

$$
Q_r = \frac{\sum_{L}C_L}{\sum_{L}C_L + \sum_{K}C_K  } 
$$

#### Qualità passante di una linea.

Nel caso di una linea in cui le macchine lavorano in serie, piuttosto che essere interessati alla qualità generale dell'insieme delle macchine, si può essere interessati alla percentuale di pezzi che attraversano l'intera linea senza essere scartati.
Se definiamo la Qualità di una macchina come la probabilità che un pezzo non venga scartato, allora la qualità complessiva di una linea si può definire come la probabilità che non venga scartato per ogni fase.
Questa definizione non fa parte dell' *OEE standard*, ma è definita come **RTY** o rolled throughput yield ed è adottata nello standard qualitativo *Six Sigma*
> una spiegazione si può trovare qui https://business.adobe.com/blog/basics/rolled-throughput-yield

$$
Q_S = \prod_r Qr = \prod_r\frac{Br}{Br + Sr}
$$

Nel caso ideale in cui una macchina riceve in input solo l'output dei pezzi conformi della macchina precendente possiamo quindi definire i buoni della macchina $r$ come la somma dei buoni e degli scarti della macchina successiva $r + 1$

$$
B_r = B_{r+1} + S{r+1}
$$ 

si può definire allora la qualità passante della linea come il rapporto tra i pezzi buoni dell'ultima macchina sull'ingresso della prima

$$
Q_s = \frac{B_f}{B_1 + S_1}
$$

oppure usando solo i buoni finali

$$
Q_s = \frac{B_f}{B_f + \sum_rS_r}
$$

ad esempio supponiamo di avere una linea composta da tre macchine. La qualità passante di questa linea può essere calcolata come:

$$
Q_s = Q_1 * Q_2 * Q_3 = \frac{B_1}{B_1 + S_1} * \frac{B_2}{B_2 + S_2} * \frac{B_3}{B_3 + S_3}
$$

possiamo definire i buoni in questo modo

$$
B_1 = B_2 + S_2
$$
$$
B_2 = B_3 + S_3
$$

andando a sostituire nella formula precedente

$$
Q_s = \frac{B_1}{B_1 + S_1} * \frac{B_2}{B_1} * \frac{B_3}{B_2} = \frac{B_3}{B_1 + S_1}
$$

o in alternativa usando solo i buoni finali

$$
Q_s =  \frac{B_3}{B_1 + S_1} = \frac{B_3}{B_2 + S_2 + S_1} = \frac{B_3}{B_3 + S_3 + S_2 + S_1}
$$


### Alternativa usando i valori calcolati in precedenza 

In maniera equivalente si può esprimere la **Availability** e la **Performance** di reparto come la somma del KPI delle singole risorse pesata per il totale del tempo di riferimento

> Questo non è possibile per la **Qualità** essendo che la qualità non ha un tempo di riferimento

#### Availability

$$
A_r = \frac{\sum_{r} (A_r * ( (S_{rf} - S_{rs} ) -\sum_{i} (P_{rie} - P_{ris}) ) ) }{\sum_{r} ( (S_{rf} - S_{rs} ) -\sum_{i} (P_{rie} - P_{ris} ) )}
$$

- $A_r$ è il valore di availability della risorsa r

- $(S_{rf} - S_{rs} ) -\sum_{i} (P_{rie} - P_{ris})$ è il tempo produttivo *previsto* della risorsa r


#### Performance

$$
P_r = \frac{\sum_{r} ( P_r * ( (S_{rf} - S_{rs} ) - \sum_{i} (P_{rie} - P_{ris}) - \sum_{j}  (U_{rje} - U_{rjs}) ) )  }{\sum_{r} ( (S_{rf} - S_{rs} ) - \sum_{i} (P_{rie} - P_{ris}) - \sum_{j}  (U_{rje} - U_{rjs}) )}
$$

- $P_r$ è il valore di performance della risorsa r

- $(S_{rf} - S_{rs} ) - \sum_{i} (P_{rie} - P_{ris}) - \sum_{j}  (U_{rje} - U_{rjs})$ è il tempo produttivo *effettivo* della risorsa r


## Differenze rispetto OEE Digital Manufacturing

- DM rispetta il calendario solo per quel che riguarda i turni: i giorni marcati come non lavorativi a calendario vengono comunque contati tra il tempo pianificato.
- DM considera per il calcolo delle performance come *tempo produttivo effettivo* la somma di tutti gli intervalli per i quali è avviato o sospeso un SFC.

> se durante un turno viene avviato un sfc solo a metà turno, e fino alla fine del turno viene mantenuta una cadenza pari al teorico, la performance per quel turno sarà del 100%
> se a metà turno viene sospeso lo sfc in corso e avviato un altro, e fino alla fine del turno viene mantenuta una cadenza pari al teorico, la performance per quel turno sarà del 75%

- DM non considera gli storni per il conteggio di pezzi prodotti

> gli storni possono comunque introdurre imprecisioni nel calcolo se avvengono al di fuori dell'unità di campionamento della conferma stornata

- DM calcola lo OEE solo per i turni conclusi.
- L'unità di campionamento più piccola per DM è il turno

> unità di campionamento più piccole per intervalli di tempo lunghi potrebbero non essere comunque disponibili per motivi di performance

