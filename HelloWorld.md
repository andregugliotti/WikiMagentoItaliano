# Hello World per Magento 1.9

## Autori

- Acierno Michele <michele.acierno@thinkopen.it>
- Luca Davide <davide.luca@thinkopen.it>
- Masto Martina <martina.masto@thinkopen.it>

## Revisori

- Gugliotti Andre <andre.gugliotti@thinkopen.it>

## Introduzione

Il modulo "Hello World" interagisce in modo basilare con i primi concetti di Magento. Fornisce all'utente tre fronti di interazione:

1. L'amministratore tramite il pannello di configurazione potrà attivare e disattivare il modulo e settare un messaggio di benvenuto.
2. Il messaggio di benvenuto, settato dall'amministratore, verrà visualizzato in tutte le pagine come elemento della colonna destra se fosse presente.
3. Il modulo, tramite il controller, offrirà una pagina di benvenuto nella quale verrà visualizzato il messaggio settato dall'amministratore.

In generale, il modulo interagisce sia per quanto riguarda la theme sia per quanto riguarda il pannello di amministrazione. A seguire vedremo come procedere con lo sviluppo di tale modulo e il significato di ogni passo.

* [CONFIGURAZIONE](#Configurazione)
* [HELPER, CONTROLLER e BLOCK](#Helper-Controller-e-Block)
* [DESIGN e LOCALE](#Design-e-Locale)

## Configurazione

### Archivio di dichiarazione del modulo
 
Per dichiarare un nuovo modulo dobbiamo creare un file xml all'interno della cartella 

    app/etc/modules/{Vendor_Module.xml}

Per esempio se avessi un path del tipo Thinkopen, che corrisponde al vendor e Special, il nome del modulo diventerà Thinkopen_Special.xml.

Il file xml sarà composto dal seguente schema:

    <?xml version="1.0" encoding="UTF-8" ?>
    <config>
        <modules>
            <Thinkopen_Special>
                <active>true</active>
                <codePool>local</codePool>
            </Thinkopen_Special>
        </modules>
    </config>

il tag config indica l'inizio dei parametri di configurazione, nella fattispecie andremo a definire i moduli (modules) e nel caso specifico il modulo che stiamo inizializzando. Il tag active e il tag codePool stanno ad indicare rispettivamente lo stato di attivazione del modulo e che tipologia (path) di module è.
In questo caso stiamo andando ad indicare la locazione del modulo. Ovvero dove quest'ultimo si trova: 
  
    app/code/local/Thinkopen/Special

Una volta definito il modulo, magento tenterà di accedere ai file di configurazione di quest'ultimo. Partendo dal path definito nel file all'interno di _etc/modules_, magento ricaverà il path per l'etc del modulo stesso: 

    app/code/local/Thinkopen/Special/etc

All'interno di questa cartella magento cercherà i file di configurazione.

### Archivi di configurazione del modulo

Nel nostro caso andremo a creare 3 file di configurazione:

- config.xml
- system.xml
- adminhtml.xml

#### config.xml

Il primo file di configurazione rappresenta il cervello, il punto nevralgico del nostro modulo.

Il file config.xml `{Vendor}/{Name}/etc/config.xml` contiene al suo interno le informazioni essenziali che permettono a Magento di comprendere dove si trovano i file, quali siano i permessi relativi al modulo, i valori di default per possibili configurazioni del modulo stesso ed altri particolati. 

Come altri file della cartella etc, oltre alla comune denominazione

    <?xml version="1.0" encoding="UTF-8" ?>

il nostro file di configurazione comincerà con il tag

    <config>
    ..
    ...
    ..
    </config>

che comunicherà a magento che gli elementi a seguire determineranno la configurazione del modulo in analisi. 

Successivamente andremo a specificare il modulo del quale stiamo parlando, difatti i tag 

    <modules>
        <thinkopen_special>
            <version>0.1.0</version>
        </thinkopen_special>
    </modules>

indicano in sequenza:

1) I moduli che andremo a trattare 
2) il tag relativo al modulo di riferimento 
3) La versione al momento dell'installazione di quest'ultimo

**Best Practice**: E' bene notare come il tag <modules> possa ospitare più di una dichiarazione, volendo potremmo dichiarare multipli moduli con multiple configurazioni. D'altronde questa pratica porterebbe ad una dispersione delle configurazioni le quali sarebbero difficili da reperire, quindi questo comportamento è **ALTAMENTE SCONSIGLIATO**. D'altro canto profilare ogni singolo file di config per i singoli moduli fornirà si una ridondanza maggiore, ma al contempo ci permetterà fasi di manutenzione, analysis e debuggin meno stressanti e complicate (Non dovremo cercare tra i centinai di file quel singolo tag che abbiamo sbagliato a scrivere! Yuppi Yeah!...)

Successivamente il tag

    <global> 
    .
    ..
    .
    </global>

andrà ad indicare lo scope di riferimento per le prossime dichiarazioni, nello specifico andremo a definire che i tag a seguire sanno condivisi in tutti gli scope di sistema.

Difatti andremo a dichiare alcuni elementi fondamentali per il nostro modulo 

     <global>
        <helpers>
            <thinkopen_special>
                <class>Thinkopen_Special_Helper</class>
            </thinkopen_special>
        </helpers>
        <blocks>
            <thinkopen_special>
                <class>Thinkopen_Special_Block</class>
            </thinkopen_special>
        </blocks>
     </global>


- helpers: gli helpers rappresentano gli "aiutanti" del modulo (come i Model ma più "deboli" per cosi dire), quest'ultimi vengono utilizzati principalmente per interazioni con i dati contenuti dal sistema (attraverso Mage o altri elementi di sistema) fornendo sostegno agli elementi del modulo.
Nello specifico andremo a dichiarare  una classe che identifica un Helper, difatti la tag class racchiude in sè proprio la classe di riferimento (e il path compresso) dei file php che saranno utilizzati come helper per il modulo.

- blocks: I block rappresentano gli elementi portanti del modulo, quest'ultimo potranno essere structural o content block (o anche ibridi). In questo caso andremo a definire una classe di riferimento che identificherà block per il nostro modulo, allo stesso modo degli helpers il nome specificato nel tag class rappresenta sia il path sia la classe.


Come è avvenuto per global ora andremo a definire quelle che sono le configurazioni nello scope (o ambito) di adminhtml, in modo similare a global anche adminihtml necessiterà di una tag apposita per specificare tale operazione

    <adminhtml>
    .
    ..
    .
    </adminhtml>
  
Al suo interno andremo a definire quindi quello che è un file di traduzione per i translate inseriti nei relativi file di configurazione di adminhtml.xml 

**NB.** Per rendere effettive queste modifiche sarà necessario selezionare nella sezione locale del pannello di amministrazione il locale relativo alla lingua prescelta, successivamente spiegheremo come è strutturata la directory locale e come posizionare i relativi file.

Ora dovremo specificare le nostre intenzioni, ovvero selezionare un file di traduzione per il nostro modulo, tali intenzioni si traducono in xml tramite i seguenti tag

       <translate>
            <modules>
                <thinkopen_special>
                    <files>
                        <default>Thinkopen_Special.csv</default>
                    </files>
                </thinkopen_special>
            </moduls>
        </translate>

Translate indica l'intenzione di specificare una traduzione e, il tag modules, indica a quali moduli esso è collegato. Infine files e default indicano quali file saranno utilizzati per la traduzione e quali di questi è il file di default per la traduzione. L'utilizzo di un csv è lo standard utilizzato nel nostro caso

Passando oltre abbiamo lo scope frontend, ovvero come il nostro modulo si interfaccerà con il frontend di magento, come prima il tag

    <frontend>
    .
    ..
    .
    </frontend>

aprirà lo scope di interesse.

Per convenzione quest'ultimo è susseguito dal tag <routers> tramite il quale andremo a specificare, per il nostro modulo, la tipologia di uso che faremo ed il frontName che verrà utilizzato per la generazione delle Urls e altre opzioni

	<routers>
        <thinkopenspecial>
            <use>standard</use>
            <args>
                <module>Thinkopen_Special</module>
                <frontName>thinkopenspecial</frontName>
            </args>
        </helloworld>
    </routers>

Il tag module andrà a specificare a quale modulo farà riferimento il frontName che andremo a scrivere, il module segue la logica del codePool selezionato nel file di configurazione all'interno di etc/modules.
Quindi se per esempio li fosse indicato il codePool: local, allora il path a cui associare il frontName sarà

    thinkopenspecial <--> app/code/local/Thinkopen/Special

Il tag layout andrà a definire, come per gli altri tag di scope, il dominio delle configurazioni che andremo a definire al suo interno. Nello specifico noi andremo a definire un file di configurazione specifico per il layout e tramite il tag updates potremo fare ciò.

	<layout>
        <updates>
            <thinkopen_special>
                <file>thinkopen_special.xml</file>
            </thinkopen_special>
        </updates>
    </layout>

il tag updates indica che è presente un possibile file di aggiuntamento per il layout relativo al nostro modulo 

    <thinkopen_special>
    .
    ..
    .
    </thinkopen_special>

il path del file <file> verrà calcolato allo stesso modo degli altri path, ovvero tramite il codePool.
Volendo potremmo specificare un file di traduzione anche per il frontend, i modi e i tag sono identici al procedimento spiegato per la sezione adminhtml, fare riferimento a quella parte della guida per tale procedura.

Infine potremmo voler definire dei valori di default per le configurazioni del nostro modulo. Tale procedura avviene proprio tramite il tag

    <default>
    .
    ..
    .
    </default>

Quest'ultimo indicherà a Magento dei valori di default che dovrà tenere in considerazione, <thinkopen_special> allo stesso modo indicherà il dominio di riferimento ed infine

    <configuration>
        <enabled>0</enabled>
        <message>Custom Message Not Defined</message>
    </configuration>

andrà a specificare che. per il gruppo configuration, i campi fileds e message avranno valori di default come indicati. 

Al termine del file config.xml, come è giusto che sia, vi sarà il tag di chiusura e, con esso, termina il file di configurazione relativo a config.xml

### adminhtml.xml

Andiamo ad analizzare il file adminhtml.xml

    <config>
        <acl>
            <resources>
                <all>
                    <title>Allow Everything</title>
                </all>
                <admin>
                    <children>
                        <system>
                            <children>
                                <config>
                                    <children>
                                        <thinkopen_special translate="title">
                                            <title>Hello World Permissions</title>
                                            <sort_order>100</sort_order>
                                        </thinkopen_special>
                                    </children>
                                </config>
                            </children>
                        </system>
                    </children>
                </admin>
            </resources>
        </acl>
    </config>

Questo particolare file di configurazione andrà a determinare la presenza di un nuovo elemento all'interno del pannello di configurazione della sezione admin di magento. Risaliamo passo per passo attraverso la struttura dell'xml per introdurre un nuovo children all'interno del config di sistema.
Il secondo file di configurazione che andiamo ad analizzare è config.xml

- config: Apre la sezione di configurazione del sistema
- acl: indica lo scope di azione delle configurazioni in atto, nel caso specifico indica che queste configurazioni agiranno a livello di pannello di amministrazione
- resources:
    - all: permette di dichiarare i permessi
    - admin...config: indica il path strutturale da seguire per l'aggiunta di un nuovo elemento nel pannello admin.
    - children: indica che il prossimo elemento sarà figlio del precedente, se non fosse presente sarà aggiunto.
    - thinkopen_special tanslate="title": indica che un nuovo elemento sarà aggiunto al pannello di amministrazione definito da noi, il titolo sarà soggetto al translate nel tag sottostante title
    - sort_order: indica l'ordine all'interno della lista. Per convenzione conviene utilizzare un multiplo di 10 affinche non si debbano spostare tutti gli elementi nel momento in cui 2 elementi o più abbiano lo stesso numero

### system.xml

Insieme a adminhtml.xml e config.xml, nella directory etc del modulo abbiamo il file di configurazione system.xml:

Si tratta di un file di configurazione, nello specifico, va a definire una nuova sezione all'interno del menù di configurazione, presente nel pannello amministratore.

system.xml agisce in System -> Configuration :: General. Troveremo la voce "Hello World" ma l'inserimento di questa voce non è compito di system.xml ma di adminhtml.xml (vedi primo paragrafo), il nostro file invece si occuperà dei dati che ci sono al suo interno. Ovvero della possibilità di abilitare e scrivere un messaggio nel pannello di amministrazione.

    <?xml version="1.0" ?>
    <config>
        <sections>
            <thinkopen_special module="thinkopen_special" translate="label">
                <label>Hello World</label>
                <tab>general</tab>
                <sort_order>60</sort_order>
                <show_in_default>1</show_in_default>
                <show_in_website>1</show_in_website>
                <show_in_store>1</show_in_store>
                <groups>
                    <configuration translate="label">
                        <label>Configuration</label>
                        <sort_order>10</sort_order>
                        <show_in_default>1</show_in_default>
                        <show_in_website>1</show_in_website>
                        <show_in_store>1</show_in_store>
                        <fields>
                            <enabled translate="label">
                                <label>Is enabled</label>
                                <sort_order>10</sort_order>
                                <tooltip>The message will appear on frontend?</tooltip>
                                <frontend_type>select</frontend_type>
                                <source_model>adminhtml/system_config_source_yesno</source_model>
                                <show_in_default>1</show_in_default>
                                <show_in_website>1</show_in_website>
                                <show_in_store>1</show_in_store>
                            </enabled>
                            <message translate="label">
                                <label>Custom message</label>
                                <sort_order>20</sort_order>
                                <tooltip>Type in the custom message to be displayed</tooltip>
                                <frontend_type>text</frontend_type>
                                <show_in_default>1</show_in_default>
                                <show_in_website>1</show_in_website>
                                <show_in_store>1</show_in_store>
                            </message>
                        </fields>
                    </configuration>
                </groups>
            </thinkopen_special>
        </sections>

Per capire il compito di questo file analizziamo prima tutte le sue parti.

    <?xml version="1.0" ?> linea iniziale per un file xml, indica la versione, a volte è indicata anche da quale tipo di string encode il nostro codice verrà trasformato
    <config> tag sempre presente se stiamo scrivendo un file di configurazione.
    <sections> indichiamo che inseriremo una o più sezioni
    <thinkopen_special module="thinkopen_special" translate="label"> in questo tag indichiamo il nostro modulo, ovvero lo scope nel quale andremo ad agire per l'introduzione della nuova sezione. Translate indica la possibilità di introdurre una traduzione tramite il token specificato in questo tag.
    <label>Hello World</label> il label, come detto precedentemente, ci permetterà di utilizzare il token per le traduzioni
    <tab>general</tab> Sotto a quale voce verranno visualizzate le nostre opzioni
    <sort_order>60</sort_order>Definisce l'ordine dell'elemento all'interno della lista di riferimento. Lo standard Magento suggerisce che quest'ultimo sia un multiplo di dieci in modo tale che anche se ci fossero più elementi con lo stesso numero di sort_order vi saranno nove altre possibilità prima di incappare in uno shift della lista. 
    
    <show_in_default>1</show_in_default>
    <show_in_website>1</show_in_website>
    <show_in_store>1</show_in_store>
    questi tre tag indicano a che livello devono essere visibili le nostre modifiche, se abbiamo 1 significa che la modifica è visibile, 0 invece indica invisibile.
    
    <groups> Sono possibili accordeon della nostra sezione di configurazione, in questo caso ne abbiamo solo uno ma possono essere anche molti.
    
    <fields> Sono i campi che andremo a compilare. 
    
    <tooltip>The message will appear on frontend?</tooltip> è un suggerimento, verrà visualizzato se sfioreremo la casella a cui è legato.
    <frontend_type>select</frontend_type> significa che il nostro elemento sarà "a tendina" ovvero che potremo scegliere un campo già impostato.
    <source_model>adminhtml/system_config_source_yesno</source_model> indica, in modo specifico, il model di riferimento per il frontend_type.

## HELPER, CONTROLLER e BLOCK

Nel nostro Modulo Hello World, ora, andremo ad inizializzare e sviluppare i vari elementi dichiarati all'interno del file di configurazione, nello specifico affronteremo per la sezione code/local:

Thinkopen/Special/Helper/Data.php
Thinkopen/Special/controllers/IndexController.php
Thinkopen/Special/Block/Special.php


### Helper/Data.php: 


    <?php
    /**
     *  Hello World
     * 
     */
    
    /**
     * Class Thinkopen_Special_IndexController
     *
     * Questo file .php rappresenta l'helper standard di Magento
     * Se non fosse definito esplicitamente alcuna classe Helper magento
     * farà riferimento e ricercherà automaticamente Data.php.
     * da qui è ben comprensibile come sia Best Practice inizializzare, anche
     * se non fosse necessario, Data.php
     *
     * Essendo un Helper, Data.php dovrà estendere la classe Mage_Core_Helper_Abstract
     * Questo avviene per due motivi principali: 
     * 1) Un oggetto di questa classe eredita i metodi del padre, potendo quindi accedere ed 
     * utilizzare l'interfaccia e i metodi di Mage_Core_Helper_Abstract
     * 2) Un oggetto helper ci consente di astrarre le operazioni meno importanti lasciando un 
     * codice più snello dove necessario e, di conseguenza, maggiormente manutenibile
     * @version 0.1.0
     * @author Think Open <www.thinkopen.it>
     */
    class Thinkopen_Special_Helper_Data extends Mage_Core_Helper_Abstract
    {
        /**
         * getConfig()
         *
         * Tramite Mage.php, e quindi il core di Magento, accediamo ad uno dei metodi 
         * portanti per ricavare le configurazioni del nostro Modulo, andando a specificare
         * il path di riferimento del nostro modulo e le configurazioni che ricerchiamo
         * @return mixed
         */
        public function getConfig($config)
        {
            return Mage::getStoreConfig("thinkopen_special/".$config);
        }
        /**
         * isEnabled()
         *
         * Tramite Mage.php, e quindi il core di Magento, accediamo ad uno dei metodi 
         * portanti per ricavare le configurazioni del nostro Modulo, andando a specificare
         * il path di riferimento del nostro modulo e le configurazioni che ricerchiamo. Nel 
         * caso specifico noi ricerchiamo la configurazione "enabled" e la restituiamo
         *
         * @return bool
         */
        public function isEnabled()
        {
            return $this->getConfig("configuration/enabled");
        }
    }

### controllers/IndexController.php

    <?php
    /**
     * Hello World Blind
     */
    
    /**
     * Class Thinkopen_Special_IndexController
     *
     * I Controller sono classi che si trovano all'interno della directory
     * /controllers/ del nostro modulo. Quest'ultimi possono essere
     * controller dedicati all'admin o al frontend. Ogni controller andrà 
     * ad estendere la classe di magento Mage_Core_Controller_Front_Action.
     * Questo comunicherà a magento che la nostra  classe è un controller e
     * che quest'ultima dovrà essere trattata come tale. 
     * Nel nostro caso ci troviamo davanti ad un controller frontend, 
     * difatti nel nostro file di configurazione  abbiamo definito un 
     * frontName, ricordate? Bene questo frontName sarà necessario per 
     * accedere ai nostri controller e, di conseguenza, ai metodi da 
     * quest'ultimo esposti. 
     * Magento attua una regola di traduzione dei path come segue: 
     * /app/{codePool}/{Vendor}/{Name}/
     * tramite il frontName ed il codePool la generazione dell'URL per le 
     * Action esposte avverrà come segue
     *
     * Site.com/frontName/class/action
     * 
     * NB. E' giusto tenere in considerazione 2 cose arrivati a questo 
     * punto: 
     * 1) Magento, identificando un controller, andrà a rimuovere i 
     * suffissi identificativi e il path della generazione del path, quindi
     * la classe IndexController per il path diverrà Index, al pari il 
     * metodo indexAction diverrà index
     * 2) Magento, trovandosi in assenza di metodi o classi specifiche, 
     * farà sempre riferimento e cercherà l'index, sia per la classe sia
     * per i metodi. Per questo motivo potremmo trovarci ad avere i 
     * seguenti link puntare alla stessa cosa.
     * 
     * Site.com/frontName
     * Site.com/frontName/index
     * Site.com/frontName/index/index
     *
     * @version 0.1.0
     * @author Think Open <www.thinkopen.it>
     */
    
    class Thinkopen_Special_IndexController extends Mage_Core_Controller_Front_Action
    {
        /**
         * Index Action
         * L'index action sarà la funzione base di riferimento per la 
         * nostra classe. Quest'ultima andrà a caricare il Layout dichiarato 
         * nel file di config e, successivamente, renderizzarlo (ovvero mostrarlo
         * a video) nella pagina. Ricordiamo che questa function corrisponde
         * ad una vera e propria pagina, riconducibile al triplice path 
         * precedentemente definiti.
         */
        public function indexAction()
        {
            $this->loadLayout();
            $this->renderLayout();
        }
    }

### Block/Thinkopenspecial.php
    
    <?php
    /**
     * Hello World Blind
     */
    
    /**
     * Class Thinkopen_Special_Block_Thinkopenspecial
     *
     * I Block rappresentano l'anima dei contenuto, quest'ultimi
     * ci permettono sia di strutturare una pagina sia di fornire i contenuti
     * necessari ai file phtml per rimpire le pagine di riferimento. 
     * I Block sono legati a doppia mandata con i file di layout e template, difatti
     * i file di layout andranno a legare i template ai blocchi.
     * 
     * Nel nostro caso, il block che stiamo andando a definire ci permette di accedere
     * alle configurazioni definite all'interno del file di configurazione tramite i 
     * metodi esposti dall'helper. 
     * Vediamo come, tramite una chiamata a magento (Mage::helper(...)) possiamo chiedere
     * a Magento un helper, che abbiamo precedentemente dichiarato nel file di configurazione.
     * Questo helper ci consentirà di accedere ai file di configurazione, qui potremmo anche 
     * modulare in un determinato modo il contenuto e, successivamente, restituirlo ai chiamanti
     * per l'utilizzo. 
     *
     * @version 0.1.0
     * @author Think Open <www.thinkopen.it>
     */
    Class Thinkopen_Special_Block_Thinkopenspecial extends Mage_Core_Block_Template
    {
        /**
         * getMessage
         *
         * Tramite Mage richiediamo l'helper dichiarato nel file di configurazione, quest'ultimo
         * esporrà i metodi ivi definiti
         * getMessage ritorna quindi il messaggio salvato come configurazione per il modulo.
         * @return mixed
         */
        public function getMessage()
        {
            return Mage::helper("thinkopen_special")->getConfig("configuration/message");
        }
    
        /**
         * isEnabled
         *
         * Tramite Mage richiediamo l'helper dichiarato nel file di configurazione, quest'ultimo
         * esporrà i metodi ivi definiti
         * isEnabled ritorna quindi lo status (disattivo o attivo intesi come false o true) salvato come configurazione per il modulo.
         * @return bool
         */
        public function isEnabled()
        {
            return Mage::helper("thinkopen_special")->isEnabledtheModule();
        }
    
    }

## DESIGN E LOCALE

### /layout/thinkopen_special.xml

Ora scendiamo più nei particolari del frontend magento. Il file di layout vanno a definire come il modulo influenzi, generi e 
modifichi le sezioni di magento ed i suoi blocchi. Essendo un file prettamente di definizione grafica e scope di influenza il tag  
iniziale sarà layout, segue un esempio 

    <?xml version="1.0" encoding="UTF-8" ?>
    <layout>
        <default>
            <reference name ="right">
                <block type ="thinkopen_special/special" name="hello.side.right" template="thinkopen_special/sidebar.phtml"/>
            </reference>
        </default>
        <thinkopenspecial_index_index>
            <reference name="root">
                <action method="setTemplate">
                    <template>page/2columns-right.phtml</template>
                </action>
            </reference>
            <reference name ="head">
                <action method="setTitle" translate="title" module="thinkopen_special"><title>Welcome to our hello!</title></action>
            </reference>
            <reference name ="content">
                <block type ="thinkopen_special/special" name="thinkopenspecial.body.content" template="thinkopen_special/content.phtml"/>
            </reference>
            <reference name ="right">
               <remove name="special.side.right"/>
            </reference>
        </thinkopenspecial_index_index>
    </layout>


il nostro tag 

    <layout>
    .
    ..
    .
    </layout> 

comunicherà a magento che il seguente blocco di tag sono relativi alle configurazioni del layout, successivamente andiamo a definire i due
scope delle modifiche da attuare, ovvero a livello generale 

    <default>
    .
    ..
    .
    </default>

e a livello di singola page definita dal controller del nostro modulo, ovvero 

    <thinopenspecial_index_index>
    .
    ..
    .
    <thinkopenspecial_index_index>

NB. Come  visto precedentemente il frontName risulta fondamentale, difatti questo tag fa proprio riferimento al nostro
frontName e definisce dove le modifiche dovranno essere applicate, nello specifico le modiche faranno riferimento al nostro
controller Index e al metodo Index di quest'ultimo. 

	<reference name ="right">
            <block type ="thinkopen_special/special" name="hello.side.right" template="thinkopen_special/sidebar.phtml"/>
        </reference>

Ora cominciamo con le modiche: il tag reference comunica a magento quale elemento della pagina dovremo andare a 
modificare tramite il parametro name (Ricordiamo che le modifiche verranno attuate sul template base che costituisce la pagina, ad esempio 
empty, o 2 columns right. Oltretutto se una reference name non fosse presente, ad esempio un template che non presenta la column right, allora
le modiche NON AVVERRANNO).

Il tag block invece andrà a linkare, rispetto alla reference che lo contiene, il Block definito in /app/local/{Vendor}/{ModuleName/Block/{Blocco linkato} con il template che dovrà essere utilizzato per il contenuto. Difatti il tag template definisce proprio un file phtml
che corrisponde ad un file html con pochi elementi php, il quale può accedere ai metodi del Block linkato e che sostanzialmente fornisce
le funzioni al template. Importante è definire anche name, UNIVOCI, per i block. Questi fungeranno da identificatori per possibili e 
future interazioni.  

Speciale menzione va al tag remove. Come detto prima possiamo interagire in più modi con il dominio in esame, volendo potremmo voler rimuovere 
un elemento dalla pagina (come nella casistica in esame). Ciò sarà possibile semplicemente definendo un remove per il name legato al block
da eliminare.

    <remove name="special.side.right"/>


**NB.** Un direttiva specifica avrà una priorità maggiore rispetto ad una direttiva più generale.
Le modifiche possono essere attuate su vari livelli: 
Singola Pagina -> Store -> Website 

### /template/thinkopen_special/content.phtml 

Segue un esempio pratico di un file phtml 

    <div class="content">
        <?php  if($this->isEnabled()) : ?>
        <?php  echo $this->getMessage(); ?>
        <?php  endif; ?>
    </div>

Il suo significato è molto semplice: utilizzeremo il template per riempire un determinato block, richiamando i metodi
definiti dal Block ad esso legato e la condizione di abilitazione per mostrare il messaggio salvato per il nostro modulo. 

### /template/thinkopen_special/sidebar.phtml 

    <?php
    if ($this->isEnabled())
    {
    echo $this->getMessage();
    }
    ?>

allo stesso modo sidebar.phtml attua un semplice controllo, prima di mostrare il messaggio salvato per il modulo. 

### /app/locale/{Formato_standard}/{Nome_del_Modulo}.csv

Ultimi, ma non ultimi, sono i file di traduzione. Quest'ultimi vengono dichiarati all'interno del file di configurazione (config.xml) e
fanno riferimento ai translate dei tag xml o alle funzioni esplicite di Magento che definiscono stringhe da tradurre. I file andranno
collocati nell'apposita cartella di rifetimento secondo il formato stardard e il formato magento.

Seguono due esempi

    /app/locale/en_US/Thinkopen_Special.csv
    /app/locale/it_IS/Thinkopen_Special.csv

I file csv seguono anche loro lo standard:

    "token","value" 
    "token2","value2"
    "token3","value3"
    "token4","value4"

Dove il token sarà la stringa, o placeholder, definita nei translate (espliciti o meno) definiti nel modulo, e il value 
sarà la traduzione associata al token per il locale settato all'interno del pannello di configurazione amministratore.