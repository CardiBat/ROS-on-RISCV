# ROS-on-RISCV

## Introduction

ROS (Acronimo di Robot Operating System) rappresenta un insieme di librerie software e strumenti destinati alla realizzazione di applicazioni robotiche. Questo sistema include una vasta gamma di componenti, che vanno dai driver agli algoritmi all'avanguardia, fino agli strumenti di sviluppo potenti, fornendo tutto il necessario per lo sviluppo di progetti robotici sofisticati. ROS 2 è stato progettato come evoluzione di ROS 1, con l'obiettivo di rispondere alle esigenze in continua evoluzione nel campo della robotica e dell'automazione, sfruttando i punti di forza di ROS 1 e migliorando gli aspetti che presentavano limitazioni.


## I primi passi

Per cominciare a capire il funzionamento di ROS e i suoi componenti principali, è possibile utilizzare varie finestre del terminale per farle successivamente interagire tra di loro. Prima di ciò, però, è importante avere chiaro i vari componenti che costituiscono il core di questo Sistema Operativo.


### Nodi

I nodi in ROS sono i componenti fondamentali, che permettono all'applicazione di funzionare correttamente. Ogni nodo è indipendente dagli altri e svolge un particolare compito che gli viene assegnato (tramite codice PYTHON). Per capire come questi nodi operano, è disponibile un pacchetto di demo chiamato 'demo_nodes_*' che ne contiene sia di semplici che di più complessi.  
Per fare qualche esempio, è disponibile un nodo di nome 'talker' che si preoccupa di mandare un messaggio di 'Hello World' ogni secondo; è inoltre disponbile un nodo chiamato 'listener' che invece si mette in ascolto di eventuali messaggi. Vi sono altri esempi più complessi, come il 'turtlesim' che rappresenta una tartaruga statica in uno spazio e il relativo nodo 'turtle_teleop_key' che manda i comandi di movimento alla tartaruga dall'input da tastiera.  
Questo insieme di nodi, ovviamente, può essere programmato ad hoc a seconda delle esigenze, prendendo magari spunto da un nodo di demo e modificato a proprio piacimento. Ne deriva inoltre che questi nodi non possono comunicare tra loro se non si adotta una certa politica di comunicazione che, come vedremo in seguito, prende il nome di 'Topic'.

### Topics

Sapendo quindi che ogni nodo è responsabile di una determinata azione, è importante chiarire il funzionamento dei topic che ne permettono la comunicazione tra essi. Quando si aprono finestre di terminale e si decide di assegnare per ognuna un nodo, la loro interazione segue tipicamente il paradigma 'pub-sub' a scambio di messaggi dove uno o più nodi fungono da publishers e altrettanti da subscribers. Quindi, i topic non sono altro che la rappresentazione di questa comunicazione.

<p>&nbsp;</p>

![Turtlesim-topic](/turtlesim.png)  

<p>&nbsp;</p>

Come si può osservare, in questo grafico (generato tramite rqt_graph con group 0 e Nodes/Topics view) si hanno i due nodi già menzionati (la tartaruga e il suo controller) e in mezzo vi sono i vari topic di comunicazione. In particolare
vi è il controller che comunica tramite topic 'cmd_vel' gli input da tastiera, mentre l'oggetto tartaruga comunica tramite due canali distinti il feedback e il suo stato attuale.  
Più nello specifico, il nodo di controllo si comporta da publisher degli input da tastiera, mentre il nodo turtlesim si iscrive a quest'ultimo topic. Allo stesso modo, turtlesim opera publishing di stato feedback e status mentre il controller si iscrive a questi ultimi.

### Servizi

I servizi in ROS rispondono all'esigenza dell'archiettura client-server il quale non è nativo per i topic così come sono stati creati. Infatti, un nodo tramite un topic classico manda un'informazione ogni x tempo ma non si aspetta una risposta dall'altra parte (come già accennato, segue il paradigma pub-sub).  
Tramite i servizi, invece, è possibile implementare ciò che caratterizza l'interazione tipicamente web in cui vi sono richieste e risposte, dei server che si preoccupano di esporre servizi e i client che ne usufruiscono.

<p>&nbsp;</p>
<div align="center">
  <img src="/services.png" alt="services.png">
</div>
<p>&nbsp;</p>

Nell'immagine soprastante vi è la rappresentazione dell'archiettura client-server in cui un client (ma potrebbero essere anche di più) manda richieste e riceve risposte tramite il servizio.  
Un esempio di come questo possa essere implementato può essere osservato sempre tramite il pacchetto demo utilizzando su un terminale add_two_ints_server (nodo Server) e in un'altra finestra la keyword 'call' aggiungendo i parametri a e b. Così facendo, si manderà una richiesta al server il quale risponderà con il risultato.


## Compilazione e Installazione

Normalmente, l'installazione (su Ubuntu) prevedere un semplice comando da terminale e l'installazione viene completata automaticamente. Nel nostro caso, però, si cercherà di compilare manualmente l'insieme di file per snellire il più possibile l'installazione e facilitarne il porting su un'architettura RISC-V (nonostante ROS sia progettato per architetture Intel e ARM)  
Quindi, partiremo in ispezione su ambiente virtuale che consenta (nel caso di errori irreversibili) di non danneggiare il sistema e di conseguenza che si possano provare varie combinazioni di compilazione. Questo passo è fondamentale per capire come muoverci agilmente quando poi ci sposteremo su RISC-V.

### Installazione di un ambiente virtuale

Prima di procedere, controllare che la propria macchina sia compatibile con la virtualizzazione integrata UNIX. Lanciare quindi il comando:

```sh
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

Se il comando restituisce un numero maggiore di 0, significa che il processore supporta la virtualizzazione. Se restituisce 0, potrebbe essere necessario abilitare la virtualizzazione nel BIOS.  
A questo punto è possibile procedere con l'installazione di KVM:  

```sh
sudo apt update
```
```sh
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

Occorre inoltre aggiungersi ai gruppi opportuni per evitare problemi:

```sh
sudo adduser `id -un` libvirtrosinstall_generator osrf_testing_tools_cpp --rosdistro foxy --deps --tar > foxy-custom.rosinstall

```
```sh
sudo adduser `id -un` kvm
```

A questo punto, dopo aver riavviato la macchina per applicare le modifiche, si può procedere a lanciare KVM e a creare una VM Ubuntu 20.04 necessaria per ROS2 Foxy, con 4Gb di RAM e 4 Cores.  
Lanciare quindi il seguente comando:

```sh
virt-manager
```

e installare il sistema operativo da file ISO scaricato dal sito Ubuntu ufficiale.


### Installazione delle librerie e dipendenze richieste da ROS2 Foxy

Sarà opportuno installare i compilatori e le librerie Python richieste, oltre a Python se non è installato sulla propria macchina. Eseguire quindi i seguenti comandi:

```sh
sudo apt update
```
```sh
sudo apt install build-essential cmake git python3-rosdep python3-rosinstall-generator python3-wstool python3-rosinstall
```

Nel caso quindi non fosse installato, per python eseguire:

```sh
sudo apt update
```
```sh
sudo apt install python3 python3-pip
```

Infine, installare il compilatore colcon:
```sh
sudo apt update
```
```sh
pip3 install -U colcon-common-extensions
```

Settare quindi permanentemente la variabile d'ambiente. Aprire la shell:
```sh
nano ~/.bashrc
```
Aggiungere quindi la seguente riga a fine file, salvare e riaprire il terminale:
```sh
export PATH="$PATH:$HOME/.local/bin"
```


### Download del Codice Sorgente di ROS

Inizializzazione `rosdep` che aiuta a installare le dipendenze di sistema per i sorgenti da compilare.

```sh
sudo rosdep init
```
```sh
rosdep update
```

### Creazione del Workspace Catkin
Per iniziare la configurazione dell'ambiente ROS, è necessaria la creazione di un workspace Catkin. Questo workspace fungerà da contenitore per i sorgenti e i pacchetti ROS. La creazione può essere effettuata con i seguenti comandi:

```sh
mkdir -p ~/ros2_foxy_ws/src
```
```sh
cd ~/ros2_foxy_ws
```

### Selezione dei Pacchetti
L'utilizzo di `rosinstall_generator` permette di selezionare specifiche parti di ROS da mettere nel file di configurazione .rosinstall. È possibile optare per l'inserimento di componenti essenziali come `ros_comm` o per una versione più completa con `desktop-full`. In ogni caso, bisogna inserire i pacchetti voluti al posto di PKGNAME. In generale quindi:

```sh
rosinstall_generator [PKGNAME] --rosdistro foxy --deps --tar > foxy-desktop.rosinstall
```
Un esempio potrebbe essere considerare pacchetti come rclcpp per la programmazione di nodi in C++, rclpy per la programmazione di nodi in Python, e example_interfaces che fornisce esempi di interfacce di servizio e messaggi che potrebbero essere utilizzati per esperimenti e apprendimento.

```sh
rosinstall_generator rclcpp rclpy example_interfaces --rosdistro foxy --deps --tar > foxy-desktop.rosinstall
```

Successivamente si scaricheranno i pacchetti presenti sul file di configurazione appena creato:

```sh
wstool init -j8 src foxy-desktop.rosinstall
```

### Compilazione di ROS
È fondamentale garantire che tutte le dipendenze dei pacchetti selezionati siano soddisfatte prima della compilazione. Questo è possibile mediante l'utilizzo di `rosdep`:

```sh
rosdep install --from-paths src --ignore-src --rosdistro foxy -y
```

Utilizzando `colcon`, lo strumento di build consigliato per ROS 2, si compilano i pacchetti scaricati con le relative dipendenze:

```sh
colcon build --symlink-install
```


### Processo di Disinstallazione e Ripristino

Per eliminare i componenti di ROS 2, è sufficiente rimuovere il workspace di compilazione, che tipicamente include le directory `build`, `install`, e `log`. Supponendo che il workspace si trovi in `~/ros2_foxy_ws`, eseguire i seguenti comandi:

```sh
cd ~/ros2_foxy_ws
```
```sh
rm -rf build install log
```
Questo rimuoverà tutte le directory relative alla compilazione e all'installazione dei pacchetti ROS 2.

Per riconfigurare l'ambiente di sviluppo da uno stato pulito, è opportuno anche rimuovere eventuali riferimenti residui a ROS 2 presenti nel file di configurazione della shell, come `.bashrc` o `.zshrc`, a seconda della shell in uso. Questi riferimenti possono includere l'aggiunta del percorso di installazione di ROS 2 al `PATH`, nonché il sourcing di script di setup.

Aprire il file di configurazione della shell con un editor di testo:

```sh
nano ~/.bashrc  # oppure utilizzare ~/.zshrc per zsh
rosinstall_generator osrf_testing_tools_cpp --rosdistro foxy --deps --tar > foxy-custom.rosinstall
```

Ricercare e rimuovere le linee associate all'ambiente ROS 2, che potrebbero apparire come:

```sh
source /opt/ros/foxy/setup.bash
```
```sh
source ~/ros2_foxy_ws/install/local_setup.bash
```

Dopo aver apportato le modifiche, salvare il file e riavviare il terminale per assicurarsi che le modifiche abbiano effetto. Questo passaggio garantisce che l'ambiente del terminale sia privo di qualsiasi configurazione relativa a ROS, permettendo una nuova installazione o la configurazione di un ambiente differente in modo pulito e senza conflitti.


## Verso il porting su RISC-V: Utilizzo del cluster Monte Cimone

Dopo aver ben chiaro il funzionamento di ROS su una macchina Ubuntu, ora si cercherà di seguire questi passaggi adattandosi alla macchina obiettivo, in questo caso basata su RISC-V. Si incontreranno diversi problemi, dal momento che non è supportata nativamente e quindi vi saranno da fare diversi interventi a livello di compilazione. Bisognerà quindi cercare di ignorare passaggi non fondamentali e di scendere a basso livello se alcuni comandi come rosdep o colcon potessero non funzionare. 

Prima di cominciare, è opportuno imparare a orientarsi con facilità all'interno della macchina RISC-V denominata Monte-Cimone. I prossimi passaggi si occuperanno di chiarire il più possibile le operazioni principali. 

### Collegamento e Login 

Per iniziare a esercitarsi sull'utilizzo di un processore di tipo RISC-V, è disponibile un cluster con vari nodi denominato Monte Cimone. Questa macchina è accessibile dal proprio PC (previa registrazione da form) tramite il comando di SSH:

```sh
ssh username@beta.dei.unibo.it -p 2223
```
> output: username@mcimone-login:~$ 


Dopo aver inserito la password fornita dal docente, cambierà il proprio username dal terminale e verrà stampato il comando di `nodeinfo` (richiamabile anche successivamente per avere informazioni in tempo reale dal `login`). La schermata mostrerà quindi tutti i nodi (disponibili _se e solo se_ sono in stato di `IDLE`):  

<p>&nbsp;</p>
<div align="center">
  <img src="/nodes.png" alt="nodes.png">
</div>
<p>&nbsp;</p>

NOTA IMPORTANTE: Il login non è basato su architettura RISC-V. Sarà quindi opportuno proseguire collegandosi a un nodo per raggiungere effettivamente l'architettura obiettivo.



### Breve descrizione dei nodi e il loro uso

I nodi disponibili sono di due tipologie a seconda della partizione. Vi sono i `sifive-nodes` i quali sono composti da 4 Core ciascuno, mentre vi sono i `milkv-nodes` che possiedono fino a 64 core per ognuno di essi. Per tutto questo sistema è installato `Slurm`, un job scheduler open source in grado di organizzare l'avvio di processi in modo semplice.  

Per entrare rapidamente in un nodo e avviare un job, è sufficiente digitare il seguente comando (con nx = n° di core):

```sh
srun -n4 --pty bash
```

Questo comando è utile per utilizzare il primo nodo disponibile, ma se invece si vuole cercare ad esempio di usare un `milkv` allora bisogna seguire il processo completo, ossia allocazione, SSH, eventuale uso e infine deallocazione.  
Per allocare un nodo si usa il comando `salloc` seguito dal n° di core e dal selettore di partizione con relativo nome. 

salloc -n64 -p mcimone-milkvs

Se non si sanno i nomi delle partizioni è sufficiente digitare il comando `sinfo` per averne l'elenco. In ogni caso, dopo questa operazione è possibile scoprire quale nodo della partizione è stato allocato con `nodeinfo` e infine eseguire il comando:

```sh
ssh fsimoni@mcimone-milkv-1
```

Riuscendo così ad entrare con Secure Shell dentro al nodo. Da qui, è possibile riutilizzare `srun` per avviare un job. Per deallocare, invece, digitare il comando `squeue` per scoprire il PID del nodo allocato e infine il comando di cancellazione seguito da quest'ultimo.

```sh
scancel [PID]
```

Per completezza, viene riportato l'utilizzo generale di questi due comandi:

```sh
salloc -n <> -t <hours:minutes:seconds> [-p <>] [-w <>] [--exclusive]
```

```sh
srun -n <n_task_to_allocate> [-N <n_nodes_to_run> ] [-p <partitions>] [-w <specific_node>] [-t <hours:minutes:seconds>] [--pty] command
```

Noi useremo il node-4 e quindi runniamo lo stesso task su 4 core:
```sh
srun -n4 -c1 -w mcimone-node-4 --pty bash
```

Per una guida più approfondita, guardare la [guida specifica di CIMONE](https://gitlab.com/ecs-lab/courses/lab-of-big-data/riscv-hpc-perf/-/blob/main/2_slurm.md?ref_type=heads) oppure la [documentazione ufficiale](https://slurm.schedmd.com/overview.html)



### Ispezione delle dipendenze base sulla macchina RISC-V

Riassumendo, le dipendenze necessarie per ROS2 sono:  

- build-essential
- cmake
- git
- python3
- python3-pip  
- python3-rosdep
- python3-rosinstall-generator
- python3-wstool
- python3-rosinstall  

Bisogna quindi verificarne la presenza sulla macchina RISC-V, eventualmente installarle e a questo punto si potrà procedere con il porting.  
_Nota importante_: Non si hanno privilegi di root per installare programmi sulla macchina che si usa, perciò eventuali programmi saranno da installare sulla propria directory Desktop da _Sorgente_ e dovrà quindi essere aggiornata la variabile d'ambiente PATH per ognuno di essi (oppure si può richiedere all'amministratore in alcuni casi).

Runniamo il seguente comando:

```sh
dpkg -l | grep <nome_pacchetto>
```

Sostituendo `<nome_pacchetto>` con python3, git, cmake e build-essential. Notando la mancanza di rosdep, rosinstall-generator, wstool e rosinstall, essi saranno da installare nella directory corrente e da aggiungere al PATH.

Installazione: 

```sh
pip3 install --user rosdep rosinstall-generator wstool rosinstall
```
Dove `--user` specifica la directory locale e non di sistema

Impostazione di PATH:

Dalla HOME (~) runnare `nano .bashrc` e aggiungere la seguente riga in fondo al file, che cerca in locale le directory dei pacchetti:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

Salvare, chiudere nano e ricaricare bashrc con il seguente comando:

```sh
source ~/.bashrc
```
Dopo un controllo generale con opzioni `--version` si noterà che tutto è stato installato completamente.


## Prove di compilazione di ROS su RISC-V

Siamo arrivati al cuore del progetto: il porting di un sistema operativo non compatibile con RISC-V. Sarà necessario muoversi con cautela stando attenti alle installazioni, ai conflitti e alle dipendenze. Inoltre, bisognerà essere in grado di trovare alternative qualora non funzionasse qualche soluzione. 

Innanzitutto, per la compilazione di ROS, si utilizza normalmente `colcon` anche se non è detto che funzioni su RISC-V. Proveremo comunque a utilizzarlo installandolo con questo comando (sempre nella directory dell'utente):

```sh
pip3 install --user -U colcon-common-extensions
```

A questo punto creare la workspace come già visto virtualmente su all'inizio:

```sh
mkdir -p ~/ros2_foxy_ws/src
```
```sh
cd ~/ros2_foxy_ws
```

### [FAIL] Prova di compilazione sui pacchetti principali ed esposizione dei problemi


Proveremo quindi ad installare due pacchetti di prova per nodi semplici di esempio e personalizzazione solo in Python3.  

File di configurazione:

```sh
rosinstall_generator rclpy example_interfaces --rosdistro foxy --deps --tar > foxy-custom.rosinstall
```

Scaricamento pacchetti:  

```sh
wstool init -j8 src foxy-custom.rosinstall
```

Ora bisogna procedere con l'installazione delle dipendenze che però possono dare problemi senza privilegi di root. Nonostante rosdep metta a disposizione il flag `--as-root=FALSE`, esso non funzionerà nel nostro caso. Dovremo quindi utilizzare rosdep senza privilegi di root poichè esso cerca in tutte le cartelle, anche quelle di admin. Per evitare che vada a cercare in cartelle su cui non ha il permesso, utilizzare questa serie di comandi:

```sh
mkdir -p $HOME/.ros/rosdep/sources.list.d
echo 'export ROSDEP_SOURCE_PATH="$HOME/.ros/rosdep/sources.list.d"' >> ~/.bashrc
source ~/.bashrc
```

e quindi avviare rosdep:
```sh
rosdep init
rosdep update
```
a questo punto si può utilizzare per risolvere le dipendenze:

```sh
rosdep install --from-paths src --ignore-src --rosdistro foxy -y
```

Ma ora ci sarà il problema di RISC-V, ovvero tentando la compilazione:
```
colcon build --symlink-install
```

L'output infatti ci dice:

```
18 packages finished [1min 37s]: 
- 1 package failed: osrf_testing_tools_cpp
- 3 packages aborted: ament_cmake_export_include_directories ament_lint_cmake ament_xmllint
- 10 packages had stderr output: ament_cmake_test ament_copyright ament_cppcheck ament_flake8 ament_lint ament_lint_cmake ament_package ament_pep257 fastcdr osrf_testing_tools_cpp
- 86 packages not processed
```

La stessa situazione si presenta anche clonando manualmente ogni dipendenza tramite comando git (con token). 

```sh
git clone https://<token>@github.com/ros2/<package>.git
```

```sh
cd rmw_implementation_cmake
colcon build --packages-select rmw_implementation_cmake
cd ..
```

Si rimane quindi bloccati in questa situazione poichè anche scaricando solo rlcpy (che ha sempre più di 100 dipendenze) la compilazione si blocca e non funziona compilare manualmente i pacchetti uno per uno perchè non si risolvono le dipendenze ugualmente. Qualsiasi nodo anche di prova dipende da rlcpp o rlcpy quindi non è comunque possibile alleggerire il carico di pacchetti che si aggira sempre sui 100.

### Possibili soluzioni e scelta della più efficace

Vi sono 3 possibili soluzioni a questi problemi:

- Uso di pacchetti precompilati
- Evitare rlcpy/rlcpp usando ad esempio pacchetti semplici di scambi di messaggi come HTTP o SOCKET quindi IPC (Internal Process Communication)
- Cambiare l'approccio del compilatore in modo manuale per ogni pacchetto che va in errore.

Le prime due sono poco interessanti per il nostro progetto. Noi vogliamo capire come compilare qualsiasi pacchetto (sempre se possibile) evitando errori intrinsechi di architettura che potrebbero trattarsi solo di semplici controlli e non di effettivo non funzionamento. *_Eviteremo quindi eventuali pacchetti di simulazione, controllo o test nel caso essi non ci permettano di proseguire_*.

Proseguiremo quindi scendendo a basso livello cambiando i file di CMAKE forzando la compilazione in tutti i casi in cui è possibile. 


## Modifica del comportamento di CMAKE a tempo di compilazione

Procederemo quindi a provare l'installazione di un pacchetto fondamentale di ROS: `rclcpp`, ovvero il package che permette la creazione di nodi C++ che, come detto all'inizio, sono la parte più importante di questo OS.

Intanto, per ricordarlo, per installare questo pacchetto basta fare:

```sh
rosinstall_generator rclcpp --rosdistro foxy --deps --tar > foxy-custom.rosinstall
```
```sh
wstool init -j8 src foxy-custom.rosinstall
```
```sh
rosdep install --from-paths src --ignore-src --rosdistro foxy -y
```
```sh
colcon build --symlink-install
```

Il quale andrà in errore come il test [FAIL] esposto sopra al 18° pacchetto.  

### Compilazione ad hoc di ogni pacchetto con eventuali modifiche

Proveremo quindi a compilare da solo osrf_testing_tools, uno dei pacchetti che fallisce e che causa `aborted` su tutti gli altri a cascata. Eseguendo quindi:

```sh
colcon build --packages-select osrf_testing_tools
```  

Si incontra un errore di questo tipo:

```
Starting >>> osrf_testing_tools_cpp
[Processing: osrf_testing_tools_cpp]                              
--- stderr: osrf_testing_tools_cpp                               
In file included from /home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/./print_backtrace.hpp:26,
                 from /home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/custom_memory_functions.cpp:30:
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3944:2: error: #warning is a GCC extension [-Werror]
 3944 | #warning ":/ sorry, ain't know no nothing none not of your architecture!"
      |  ^~~~~~~
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3944:2: error: #warning ":/ sorry, ain't know no nothing none not of your architecture!" [-Werror=cpp]
In file included from /home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/./stack_trace_impl.hpp:31,
                 from /home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/memory_tools_service.cpp:20:
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3944:2: error: #warning is a GCC extension [-Werror]
 3944 | #warning ":/ sorry, ain't know no nothing none not of your architecture!"
      |  ^~~~~~~
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3944:2: error: #warning ":/ sorry, ain't know no nothing none not of your architecture!" [-Werror=cpp]
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp: In static member function ‘static void backward::SignalHandling::handleSignal(int, siginfo_t*, void*)’:
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3920:17: error: unused variable ‘uctx’ [-Werror=unused-variable]
 3920 |     ucontext_t *uctx = static_cast<ucontext_t *>(_ctx);
      |                 ^~~~
cc1plus: all warnings being treated as errors
gmake[2]: *** [src/memory_tools/CMakeFiles/memory_tools.dir/build.make:76: src/memory_tools/CMakeFiles/memory_tools.dir/custom_memory_functions.cpp.o] Error 1
gmake[2]: *** Waiting for unfinished jobs....
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp: In static member function ‘static void backward::SignalHandling::handleSignal(int, siginfo_t*, void*)’:
/home/fsimoni/ros2_foxy_ws/src/osrf_testing_tools_cpp/src/memory_tools/././vendor/bombela/backward-cpp/backward.hpp:3920:17: error: unused variable ‘uctx’ [-Werror=unused-variable]
 3920 |     ucontext_t *uctx = static_cast<ucontext_t *>(_ctx);
      |                 ^~~~
In file included from /home/fsimoni/ros2_foxy_ws/build/osrf_testing_tools_cpp/googletest-1.10.0.1-extracted/googletest-1.10.0.1-src/googletest/src/gtest-all.cc:42:
/home/fsimoni/ros2_foxy_ws/build/osrf_testing_tools_cpp/googletest-1.10.0.1-extracted/googletest-1.10.0.1-src/googletest/src/gtest-death-test.cc: In function ‘bool testing::internal::StackGrowsDown()’:
/home/fsimoni/ros2_foxy_ws/build/osrf_testing_tools_cpp/googletest-1.10.0.1-extracted/googletest-1.10.0.1-src/googletest/src/gtest-death-test.cc:1301:24: error: ‘dummy’ may be used uninitialized [-Werror=maybe-uninitialized]
 1301 |   StackLowerThanAddress(&dummy, &result);
      |   ~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~
/home/fsimoni/ros2_foxy_ws/build/osrf_testing_tools_cpp/googletest-1.10.0.1-extracted/googletest-1.10.0.1-src/googletest/src/gtest-death-test.cc:1290:13: note: by argument 1 of type ‘const void*’ to ‘void testing::internal::StackLowerThanAddress(const void*, bool*)’ declared here
 1290 | static void StackLowerThanAddress(const void* ptr, bool* result) {
      |             ^~~~~~~~~~~~~~~~~~~~~
/home/fsimoni/ros2_foxy_ws/build/osrf_testing_tools_cpp/googletest-1.10.0.1-extracted/googletest-1.10.0.1-src/googletest/src/gtest-death-test.cc:1299:7: note: ‘dummy’ declared here
 1299 |   int dummy;
      |       ^~~~~
cc1plus: all warnings being treated as errors
gmake[2]: *** [src/memory_tools/CMakeFiles/memory_tools.dir/build.make:132: src/memory_tools/CMakeFiles/memory_tools.dir/memory_tools_service.cpp.o] Error 1
gmake[1]: *** [CMakeFiles/Makefile2:1012: src/memory_tools/CMakeFiles/memory_tools.dir/all] Error 2
gmake[1]: *** Waiting for unfinished jobs....
cc1plus: all warnings being treated as errors
gmake[2]: *** [googletest-1.10.0.1-extracted/googletest-1.10.0.1-build/googletest/CMakeFiles/gtest.dir/build.make:76: googletest-1.10.0.1-extracted/googletest-1.10.0.1-build/googletest/CMakeFiles/gtest.dir/src/gtest-all.cc.o] Error 1
gmake[1]: *** [CMakeFiles/Makefile2:1143: googletest-1.10.0.1-extracted/googletest-1.10.0.1-build/googletest/CMakeFiles/gtest.dir/all] Error 2
gmake: *** [Makefile:146: all] Error 2
---
Failed   <<< osrf_testing_tools_cpp [47.4s, exited with code 2]

Summary: 0 packages finished [50.0s]
  1 package failed: osrf_testing_tools_cpp
  1 package had stderr output: osrf_testing_tools_cpp


```

Si procede quindi a ignorare i warning poichè non interessanti e a sopprimere da txt quelli dell'architettura. Si noti infatti che questi errori non sono fatali ma bensì semplici warning trattati come errori su RISC-V. _Si farà lo stesso per gli altri pacchetti a seguire prestando attenzione a non comprometterli_.:

```sh
nano src/osrf_testing_tools_cpp/src/memory_tools/vendor/bombela/backward-cpp/backward.hpp
```

rimuovere quindi `#warning ":/ sorry, ain't know no nothing none not of your architecture!"` e runnare:

```sh
colcon build --cmake-args -DCMAKE_CXX_FLAGS="-Wno-error -Wno-unused-variable -Wno-maybe-uninitialized -Wno-error=cpp -Wno-error=pedantic"
```

Procedendo, purtroppo vi sono altre tipologie di errori per molti pacchetti, tutti dovuti alla mancanza di un pacchetto 'benchmark'.

```
CMake Error at CMakeLists.txt:20 (find_package):
  By not providing "Findbenchmark.cmake" in CMAKE_MODULE_PATH this project
  has asked CMake to find a package configuration file provided by
  "benchmark", but CMake did not find one.

  Could not find a package configuration file provided by "benchmark" with
  any of the following names:

    benchmarkConfig.cmake
    benchmark-config.cmake

  Add the installation prefix of "benchmark" to CMAKE_PREFIX_PATH or set
  "benchmark_DIR" to a directory containing one of the above files.  If
  "benchmark" provides a separate development package or SDK, be sure it has
  been installed.


---
Failed   <<< performance_test_fixture [6.56s, exited with code 1]
Aborted  <<< rosidl_generator_dds_idl [7.96s]                                                                                                                                      
Aborted  <<< rosidl_typesupport_interface [1min 50s]                                                                                         
Aborted  <<< fastrtps [16min 1s]                                             

Summary: 60 packages finished [20min 13s]
  1 package failed: performance_test_fixture
  3 packages aborted: fastrtps rosidl_generator_dds_idl rosidl_typesupport_interface
  23 packages had stderr output: ament_cmake ament_cmake_cppcheck ament_cmake_cpplint ament_cmake_flake8 ament_cmake_gmock ament_cmake_google_benchmark ament_cmake_gtest ament_cmake_pep257 ament_cmake_pytest ament_cmake_ros ament_cmake_uncrustify ament_cmake_xmllint ament_lint_auto ament_lint_common fastrtps foonathan_memory_vendor performance_test_fixture python_cmake_module rosidl_adapter rosidl_cmake rosidl_generator_dds_idl rosidl_parser test_interface_files
  44 packages not processed
```

Seguiamo quindi il suggerimento `Add the installation prefix of "benchmark" to CMAKE_PREFIX_PATH`, ricordando che non si hanno privilegi di root.   

Clonare la repo in locale per evitare il comando `sudo`
```sh
git clone https://github.com/google/benchmark.git
```
```sh
cd benchmark
```
```sh
mkdir build
```
```sh
cd build
```

Disabilitare infine i test di Google per evitare errori inattesi successivi:
```sh
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/local -DBENCHMARK_ENABLE_TESTING=OFF ..
```
E procedere con l'installazione di `benchmark`
```sh
make
```
```sh
make install
```

A questo punto, però, si presenta un errore importante durante la compilazione (dato dal pacchetto `mimick`):
```
CMake Error at CMakeLists.txt:88 (message):
  Architecture 'riscv64' is not supported.
```

Per fortuna, questo pacchetto non è essenziale per la costruzione dei nodi bensì in ambito di simulazione e quindi, può essere ignorato tramite la modifica del comando di compilazione (da qui in avanti usato per pacchetti non importanti che danno errori di questo tipo):

```sh
colcon build --packages-skip mimick_vendor
```

Ovviamente vi sono altri pacchetti che dipendono da `mimick_vendor` e per ognuno bisogna entrare nel suo file di configurazione CMakeLists.txt e ignorarne la dipendenza. Vediamo un esempio per rcutils:

```sh
cd src/rcutils && nano CMakeLists.txt
```

e commentare la riga find_package(mimick_vendor REQUIRED) tramite `#`.  

Purtroppo però le dipendenze di questo pacchetto si presentano in molti file che lo utilizzano per testing. Dopo un'attenta ricerca, il comando per evitare ciò diventa:

```sh
colcon build --packages-skip mimick_vendor --cmake-args -DBUILD_TESTING=OFF
```

Di fatti, questo comando ci permette di evitare tutti gli altri pacchetti di test che non sono molto interessanti nel nostro caso e potrebbero portare a una grande perdita di tempo.  

A questo punto la compilazione prosegue fino al pacchetto finale di installazione rclcpp che però va in errore in questo modo:

```
[Processing: rclcpp]                                             
--- stderr: rclcpp                                              
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/executor.cpp: In member function ‘bool rclcpp::Executor::get_next_ready_executable(rclcpp::AnyExecutable&)’:
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/executor.cpp:615:67: warning: ‘using CallbackGroupType = enum class rclcpp::CallbackGroupType’ is deprecated: use rclcpp::CallbackGroupType instead [-Wdeprecated-declarations]
  615 |       any_executable.callback_group->type() == CallbackGroupType::MutuallyExclusive)
      |                                                                   ^~~~~~~~~~~~~~~~~
In file included from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/any_executable.hpp:20,
                 from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/memory_strategy.hpp:24,
                 from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/memory_strategies.hpp:18,
                 from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/executor_options.hpp:20,
                 from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/executor.hpp:33,
                 from /home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/executor.cpp:24:
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/include/rclcpp/callback_group.hpp:165:7: note: declared here
  165 | using CallbackGroupType [[deprecated("use rclcpp::CallbackGroupType instead")]] = CallbackGroupType;
      |       ^~~~~~~~~~~~~~~~~
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/parameter_value.cpp: In instantiation of ‘std::string array_to_string(const std::vector<ValType>&, std::ios_base::fmtflags) [with ValType = std::__cxx11::basic_string<char>; PrintType = std::__cxx11::basic_string<char>; std::string = std::__cxx11::basic_string<char>; std::ios_base::fmtflags = std::ios_base::fmtflags]’:
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/parameter_value.cpp:105:29:   required from here
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/parameter_value.cpp:70:22: warning: loop variable ‘value’ creates a copy from type ‘const std::__cxx11::basic_string<char>’ [-Wrange-loop-construct]
   70 |   for (const ValType value : array) {
      |                      ^~~~~
/home/fsimoni/ros2_foxy_ws/src/rclcpp/rclcpp/src/rclcpp/parameter_value.cpp:70:22: note: use reference type to prevent copying
   70 |   for (const ValType value : array) {
      |                      ^~~~~
      |                      &
---
Finished <<< rclcpp [5min 5s]
                               
Summary: 107 packages finished [14min 2s]
  1 package had stderr output: rclcpp


```

Esaminando lo stderr, notiamo che abbiamo due warning:
- Un warning relativo a una dichiarazione deprecata che andrebbe aggiornata, ma non è un problema critico;
- Un warning relativo a un ciclo di while che abbassa la performance, ma anche qui non è interessante per noi.
Di conseguenza il pacchetto è quindi stato compilato completamente e si può procedere con la demo sui nodi.


## Creazione di Nodi tramite ROS installato su RISC-V

Siamo quindi riusciti a installare quasi per intero la libreria `rclcpp` con tutte le sue dipendenze. Dobbiamo quindi cercare di creare nodi C++ funzionanti per dimostrarne il funzionamento.

### [NOTA IMPORTANTE] Aggiunta ricorsiva delle variabili al PATH per ogni pacchetto

Ad ogni nuova sessione è necessario il sourcing-export, quindi eseguire questi 3 comandi:

```sh
source ~/.bashrc
export CPLUS_INCLUDE_PATH=$(find /home/fsimoni/ros2_foxy_ws/install -type d -name include | paste -sd ":" -):${CPLUS_INCLUDE_PATH}
export LD_LIBRARY_PATH=$(find /home/fsimoni/ros2_foxy_ws/install -type d -name lib | paste -sd ":" -):${LD_LIBRARY_PATH}
```


### Creazione di nodi semplici di esempio

Per creare nodi in C++ è necessario creare un pacchetto che contenga il file da compilare, il quale chiama le librerie dipendenti da `rclcpp` all'occorrenza, già installate nel capitolo precedente. Faremo due prove creando due pacchetti diversi, uno semplice e uno più complesso. Per ognuno, sarà creato un nodo ed eseguito.  

Spostiamoci in src e creiamo il nostro pacchetto:

```sh
cd /path/to/ros2_foxy_ws/src
mkdir my_package
cd my_package
```

Creiamo un file di main.cpp da cui avvieremo il nodo:
```sh
nano main.cpp
```

Scriviamone un semplice di prova, fatto in questo modo:

```
#include "rclcpp/rclcpp.hpp"

class MyNode : public rclcpp::Node
{
public:
  MyNode() : Node("my_node") {
    RCLCPP_INFO(this->get_logger(), "Hello from my_node!");
  }
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<MyNode>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

Salviamolo e compiliamolo singolarmente, dal root del progetto:

```sh
g++ -o my_node src/my_package/main.cpp \
-I/home/fsimoni/ros2_foxy_ws/install/rclcpp/include \
-I/home/fsimoni/ros2_foxy_ws/install/rcutils/include \
/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib/librclcpp.so \
/home/fsimoni/ros2_foxy_ws/install/rcutils/lib/librcutils.so \
-lstdc++fs -pthread
```

Da notare che sono state specificate a tempo di compilazione le librerie che non sono state aggiunte al PATH per evitare confusione. Di fatti, senza queste specifiche vi sarebbero una serie di errori. Dopo una lunga ricerca, abbiamo quindi capito come specificarle.

Verrà quindi creato un eseguibile chiamato my_node. Runniamolo in modo classico:

```sh
./my_node
```
Il nodo funziona e stampa a terminale:

<p>&nbsp;</p>
<div align="center">
  <img src="/node1.png" alt="node1.png">
</div>
<p>&nbsp;</p>

Per sicurezza, effettuiamo un backup:

```sh
scp -r fsimoni@mcimone-node-4:/home/fsimoni/ros2_foxy_ws /home/cardigun/Scrivania/backup-riscv
```

Creiamo un altro nodo di esempio. Creiamo quindi un `my_package_2` in `src` e dentro creo un altro nodo così composto:

```
#include "rclcpp/rclcpp.hpp"

class MyNode2 : public rclcpp::Node
{
public:
  MyNode2() : Node("my_node_2") {
    // Do something different here
    timer_ = this->create_wall_timer(std::chrono::seconds(1), std::bind(&MyNode2::timerCallback, this));
    RCLCPP_INFO(this->get_logger(), "Hello from my_node_2!");
  }

private:
  void timerCallback() {
    // Perform some action periodically
    RCLCPP_INFO(this->get_logger(), "Doing something different...");
  }

  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<MyNode2>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}

```

Compiliamo quindi in modo simile ma aggiungendo le librerie che servono:

```sh
g++ -o my_node_2 src/my_package_2/main.cpp \
-I/home/fsimoni/ros2_foxy_ws/install/rclcpp/include \
-I/home/fsimoni/ros2_foxy_ws/install/rcutils/include \
/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib/librclcpp.so \
/home/fsimoni/ros2_foxy_ws/install/rcutils/lib/librcutils.so \
/home/fsimoni/ros2_foxy_ws/install/rcl/lib/librcl.so \
/home/fsimoni/ros2_foxy_ws/install/tracetools/lib/libtracetools.so \
-lstdc++fs -pthread
```

E runniamo:

```sh
./my_node_2
```

Ad output quindi si mostra un nodo in grado di poter rimanere in ascolto di eventuali segnali di sensori provenienti dall'esterno. Potrebbe essere quindi in grado di eseguire un `handler` nel caso arrivi un `interrupt`. :

<p>&nbsp;</p>
<div align="center">
  <img src="/node2.png" alt="node2.png">
</div>
<p>&nbsp;</p>


### Creazione di nodi complessi e comunicanti tramite Topic (pub-sub)

Creiamo un nodo 'client' (publisher) così composto:

```
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"

class ClientNode : public rclcpp::Node
{
public:
    ClientNode() : Node("client_node")
    {
    	// creating publisher
        publisher_ = this->create_publisher<std_msgs::msg::Int32>("server_topic", 10);
        
        // wall timer doing callback every 5s
        timer_ = this->create_wall_timer(
            std::chrono::seconds(5), 
            std::bind(&ClientNode::send_message, this) 
        );
	
	// start log
        RCLCPP_INFO(this->get_logger(), "Client node started.");
    }

private:
    void send_message()
    {
    	// sending messages (2 types)
        auto message = std_msgs::msg::Int32();
        message.data = 1; // switch 1 or 2 --> HW connected if needed (KEYBOARD)
        
        RCLCPP_INFO(this->get_logger(), "Info requested: %d", message.data);
        publisher_->publish(message);
    }

    rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<ClientNode>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```
Questo nodo ha due modalità selezionabili:
- In modalità 1 richiede la temperatura esterna (publish) sul topic
- In modalità 2 richiede la pressione esterna (publish) sul topic  

Compilabile con:

```
g++ -o client src/my_package_4/client.cpp -I/home/fsimoni/ros2_foxy_ws/install/rclcpp/include -I/home/fsimoni/ros2_foxy_ws/install/rcutils/include -I/home/fsimoni/ros2_foxy_ws/install/rcl/include -I/home/fsimoni/ros2_foxy_ws/install/tracetools/include -I/home/fsimoni/ros2_foxy_ws/install/rmw/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/include -I/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/include -I/home/fsimoni/ros2_foxy_ws/install/rcpputils/include -I/home/fsimoni/ros2_foxy_ws/install/std_msgs/include -I/home/fsimoni/ros2_foxy_ws/install/builtin_interfaces/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_cpp/include -I/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/include -L/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib -L/home/fsimoni/ros2_foxy_ws/install/rcutils/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl/lib -L/home/fsimoni/ros2_foxy_ws/install/tracetools/lib -L/home/fsimoni/ros2_foxy_ws/install/rmw/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/lib -L/home/fsimoni/ros2_foxy_ws/install/rcpputils/lib -L/home/fsimoni/ros2_foxy_ws/install/statistics_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/lib -L/home/fsimoni/ros2_foxy_ws/install/std_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rosgraph_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_cpp/lib -L/home/fsimoni/ros2_foxy_ws/install/libstatistics_collector/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcutils/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/tracetools/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rmw/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcpputils/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/statistics_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/std_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosgraph_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_cpp/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/libstatistics_collector/lib -lrclcpp -lrcutils -lrcl -ltracetools -lrmw -lrosidl_runtime_c -lrcl_yaml_param_parser -lrcpputils -lstatistics_msgs__rosidl_typesupport_cpp -lstd_msgs__rosidl_typesupport_cpp -lrosgraph_msgs__rosidl_typesupport_cpp -lrosidl_typesupport_cpp -lstdc++fs -pthread -llibstatistics_collector
```

Creiamo anche il nodo server (subscriber):

```
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/int32.hpp>

class ServerNode : public rclcpp::Node
{
public:
    ServerNode() : Node("server_node")
    {
        subscription_ = this->create_subscription<std_msgs::msg::Int32>(
            "server_topic", 10, std::bind(&ServerNode::handle_message, this, std::placeholders::_1));
    }

private:
    void handle_message(const std_msgs::msg::Int32::SharedPtr msg)
    {
        int result = 0;
        if (msg->data == 1)
        {
            result = function1();
        }
        else if (msg->data == 2)
        {
            result = function2();
        }
        else
        {
            RCLCPP_WARN(this->get_logger(), "Received unsupported value: %d", msg->data);
            return;
        }

        RCLCPP_INFO(this->get_logger(), "Function result: %d", result);
    }

    int function1()
    {
        RCLCPP_INFO(this->get_logger(), "Executing function1");
        return 10; // valore di ritorno di esempio
    }

    int function2()
    {
        RCLCPP_INFO(this->get_logger(), "Executing function2");
        return 20; // valore di ritorno di esempio
    }

    rclcpp::Subscription<std_msgs::msg::Int32>::SharedPtr subscription_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<ServerNode>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```
Questo nodo riceve messaggi dal nodo publisher e a seconda della tipologia risponde:
- 1 --> temperatura
- 2 --> pressione




Compilabile con:

```
g++ -o server src/my_package_4/server.cpp -I/home/fsimoni/ros2_foxy_ws/install/rclcpp/include -I/home/fsimoni/ros2_foxy_ws/install/rcutils/include -I/home/fsimoni/ros2_foxy_ws/install/rcl/include -I/home/fsimoni/ros2_foxy_ws/install/tracetools/include -I/home/fsimoni/ros2_foxy_ws/install/rmw/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/include -I/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/include -I/home/fsimoni/ros2_foxy_ws/install/rcpputils/include -I/home/fsimoni/ros2_foxy_ws/install/std_msgs/include -I/home/fsimoni/ros2_foxy_ws/install/builtin_interfaces/include -I/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_cpp/include -I/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/include -L/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib -L/home/fsimoni/ros2_foxy_ws/install/rcutils/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl/lib -L/home/fsimoni/ros2_foxy_ws/install/tracetools/lib -L/home/fsimoni/ros2_foxy_ws/install/rmw/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/lib -L/home/fsimoni/ros2_foxy_ws/install/rcpputils/lib -L/home/fsimoni/ros2_foxy_ws/install/statistics_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/lib -L/home/fsimoni/ros2_foxy_ws/install/std_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rosgraph_msgs/lib -L/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_cpp/lib -L/home/fsimoni/ros2_foxy_ws/install/libstatistics_collector/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rclcpp/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcutils/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/tracetools/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rmw/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_runtime_c/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_interface/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl_yaml_param_parser/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcpputils/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/statistics_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rcl_interfaces/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/std_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosgraph_msgs/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/rosidl_typesupport_cpp/lib -Wl,-rpath,/home/fsimoni/ros2_foxy_ws/install/libstatistics_collector/lib -lrclcpp -lrcutils -lrcl -ltracetools -lrmw -lrosidl_runtime_c -lrcl_yaml_param_parser -lrcpputils -lstatistics_msgs__rosidl_typesupport_cpp -lstd_msgs__rosidl_typesupport_cpp -lrosgraph_msgs__rosidl_typesupport_cpp -lrosidl_typesupport_cpp -lstdc++fs -pthread -llibstatistics_collector
```

Qui sotto vi è l'immagine che dimostra l'interazione tra essi:

<p>&nbsp;</p>
<div align="center">
  <img src="/nodi_comunicanti.png" alt="nodi_comunicanti.png">
</div>
<p>&nbsp;</p>

Si dimostra quindi come la comunicazione tra i nodi in RISC-V avviene senza problemi e, ovviamente, è estendibile in senso HW collegando ad esempio il nodo Server a sensori di temperatura/pressione. A sua volta, il nodo client può essere collegato a uno switch fisico per il cambio di modalità (o pre-programmato). Si noti infine che questo paradigma non è propriamente client-server ma è pub-sub, il che significa che le risposte del server necessitano di essere lette su altri nodi nel caso ci si voglia comportare di conseguenza nell'applicazione (ad esempio, innalzando la temperatura con dispositivi appropriati collegati ai nuovi nodi).


### [NOTA]: 
Il motivo per il quale ad ogni compilazione aggiungiamo i vari percorsi è perchè non abbiamo installato ROS2 su x86/ARM e quindi non sono disponibili classici comandi come:

```sh
ros2 run my_package_2 my_node_2
```

Di fatti `ros2` non verrebbe trovato. Specifico quindi a compilazione dove cercare le librerie compilate e le altre esterne necessarie in modo tale di non incontrare problemi se utilizzo `./exec` per il run, nativo per g++. 


## Analisi finali

Si rende ora necessario poter monitorare le risorse richieste durante l'esecuzione e la compilazione (per un futuro export su FPGA ad esempio). Per non dover ricorrere ad altre installazioni esterne alla macchina (che potrebbero portare via molto tempo e potrebbero risultare incompatibili), si utilizza htop dopo le run per poter monitorare queste informazioni. 

Le colonne che ci interessano sono:

- Utilizzo della CPU (%CPU): Verifica se il processo richiede molto tempo di CPU.
- Memoria Residente (RES): La quantità di RAM effettivamente utilizzata dal processo.

Dopo aver runnato entrambi i nodi, ricercarli tramite filtro:

- Premere f4 (filter)
- Incollare (Ctrl-alt-V) il seguente filtro dei nodi: client|server. Premere quindi invio.

_Ricordiamoci infine che la macchina che stiamo utilizzando ha 4 core SiFive U74-MC da 1.5GHz di clock e 16GB di RAM_

### Benchmarking delle risorse a run-time al variare dell'intensità di comunicazione

Per poter ottenere informazioni sulle risorse necessarie a tempo di esecuzione del programma, si rende necessaria la modifica del codice in modo tale che sia possibile cambiare la frequenza di invio di messaggi durante la run (e non dover ricompilare ogni volta). Si modifica quindi il nodo publisher rispettando questa specifica (ed evitando di stampare a terminale in entrambi i nodi).

```
public:
    ClientNode(int publish_frequency_ms) : Node("client_node")
    {
        // Creare il publisher
        publisher_ = this->create_publisher<std_msgs::msg::Int32>("server_topic", 10);

        // Creare il timer con la frequenza configurabile
        timer_ = this->create_wall_timer(
            std::chrono::milliseconds(publish_frequency_ms), 
            std::bind(&ClientNode::send_message, this)
        );

        // Iniziare log
        RCLCPP_INFO(this->get_logger(), "Client node started with frequency: %d ms", publish_frequency_ms);
    }

private:
    void send_message()
    {
        // Inviare messaggi (2 tipi)
        auto message = std_msgs::msg::Int32();
        message.data = 1; // Cambiare a 2 se necessario

        publisher_->publish(message);
    }

    rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);

    // Controlla se    stato passato un argomento
    if (argc < 2) {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Usage: client_node <publish_frequency_ms>");
        return 1;
    }

    // Converti l'argomento in intero
    int publish_frequency_ms = std::atoi(argv[1]);

    // Verifica che l'argomento sia valido
    if (publish_frequency_ms <= 0) {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Invalid publish frequency: %d ms", publish_frequency_ms);
        return 1;
    }

    // Creare il nodo con la frequenza di pubblicazione specificata
    auto node = std::make_shared<ClientNode>(publish_frequency_ms);

    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}


```

Il nodo server invece stampa su file per controllare che vada tutto ok:
```
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/int32.hpp>
#include <fstream>

class ServerNode : public rclcpp::Node
{
public:
    ServerNode(std::shared_ptr<std::ofstream> log_file) 
        : Node("server_node"), log_file_(log_file)
    {
        subscription_ = this->create_subscription<std_msgs::msg::Int32>(
            "server_topic", 10, std::bind(&ServerNode::handle_message, this, std::placeholders::_1));
    }

private:
    void handle_message(const std_msgs::msg::Int32::SharedPtr msg)
    {
        int result = 0;
        if (msg->data == 1)
        {
            result = function1();
        }
        else if (msg->data == 2)
        {
            result = function2();
        }
        else
        {
            *log_file_ << "Received unsupported value: " << msg->data << std::endl;
            return;
        }

        *log_file_ << "Function result: " << result << std::endl;
    }

    int function1()
    {
        *log_file_ << "Executing function1" << std::endl;
        return 10; // valore di ritorno di esempio
    }

    int function2()
    {
        *log_file_ << "Executing function2" << std::endl;
        return 20; // valore di ritorno di esempio
    }

    rclcpp::Subscription<std_msgs::msg::Int32>::SharedPtr subscription_;
    std::shared_ptr<std::ofstream> log_file_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);

    // Apri il file di log
    auto log_file = std::make_shared<std::ofstream>("/home/fsimoni/ros2_foxy_ws/output.txt", std::ios_base::app);

    auto node = std::make_shared<ServerNode>(log_file);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}

```



A questo punto, dopo aver compilato, è possibile runnare il nodo publisher passando il parametro di frequenza (s) con 5ms = circa 200 messaggi al secondo

```sh
./client_benchmark 5 &
```

Dopo aver avviato anche il subscriber e quindi l'interazione è cominciata, si può vedere a terminale la gestione delle risorse specifica per questi due nodi tramite il comando htop personalizzato che trova i pid dei processi e monitora solo loro:

```sh
htop -p $(pgrep -d',' -f 'client_benchmark|server_benchmark')
```

Oppure, per essere più accurati, si può runnare (per la RAM):
```sh
ps aux | grep -E 'client_benchmark|server_benchmark' | grep -v grep | awk '{sum += $6} END {print sum/1024 " MB"}'
```
E per la CPU
```sh
ps aux | grep -E 'client_benchmark|server_benchmark' | grep -v grep | awk '{sum += $3} END {print sum " %"}'
```



Da notare che conviene, a tempo di compilazione, dare nomi specifici ai nodi da runnare per evitare che il comando di ps ricerchi altri processi con nomi simili.  

Prima di procedere con il benchmarking, bisogna scegliere le frequenze di scambi di messaggi da testare. Normalmente, un robot mobile possiede questa configurazione:

- LiDAR: 2 sensori a 10 Hz, 0.05MB ogni messaggio
- Camera: 2 camere a 30 Hz, 1.5MB ogni messaggio
- Inertial Measurement Unit (IMU): 1 sensore a 100 Hz, 0.0005MB ogni messaggio
- Sensori di distanza: 4 sensori a 10 Hz, 0.0005MB ogni messaggio
- Sensori di temperatura: 2 sensori a 1 Hz, 0.0005MB ogni messaggio

Di conseguenza vengono circa scambiati 220 messaggi al secondo, con un peso totale di circa 90MB al secondo. Per semplificare, possiamo quindi dire che ogni messaggio peserebbe circa 0.4MB.  

Si utilizzeranno quindi altri 2 nodi modificati sulla base di queste specifiche:

client
```
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include <cstdlib>
#include <string>

class ClientNode : public rclcpp::Node
{
public:
    ClientNode(int publish_frequency_ms) : Node("client_node")
    {
        // Creare il publisher
        publisher_ = this->create_publisher<std_msgs::msg::String>("server_topic", 10);

        // Creare il timer con la frequenza configurabile
        timer_ = this->create_wall_timer(
            std::chrono::milliseconds(publish_frequency_ms), 
            std::bind(&ClientNode::send_message, this)
        );

        // Iniziare log
        RCLCPP_INFO(this->get_logger(), "Client node started with frequency: %d ms", publish_frequency_ms);
    }

private:
    void send_message()
    {
        // Creare un messaggio di 0.4 MB
        auto message = std_msgs::msg::String();
        message.data = std::string(400 * 1024, '0'); // Stringa di 400 KB

        // Pubblicare il messaggio
        publisher_->publish(message);
    }

    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);

    // Controlla se    stato passato un argomento
    if (argc < 2) {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Usage: client_node <publish_frequency_ms>");
        return 1;
    }

    // Converti l'argomento in intero
    int publish_frequency_ms = std::atoi(argv[1]);

    // Verifica che l'argomento sia valido
    if (publish_frequency_ms <= 0) {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Invalid publish frequency: %d ms", publish_frequency_ms);
        return 1;
    }

    // Creare il nodo con la frequenza di pubblicazione specificata
    auto node = std::make_shared<ClientNode>(publish_frequency_ms);

    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

server
```
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/int32.hpp>
#include <std_msgs/msg/string.hpp>
#include <fstream>

class ServerNode : public rclcpp::Node
{
public:
    ServerNode(std::shared_ptr<std::ofstream> log_file) 
        : Node("server_node"), log_file_(log_file)
    {
        subscription_int_ = this->create_subscription<std_msgs::msg::Int32>(
            "server_topic", 10, std::bind(&ServerNode::handle_int_message, this, std::placeholders::_1));

        subscription_string_ = this->create_subscription<std_msgs::msg::String>(
            "server_topic", 10, std::bind(&ServerNode::handle_string_message, this, std::placeholders::_1));
    }

private:
    void handle_int_message(const std_msgs::msg::Int32::SharedPtr msg)
    {
        int result = 0;
        if (msg->data == 1)
        {
            result = function1();
        }
        else if (msg->data == 2)
        {
            result = function2();
        }
        else
        {
            *log_file_ << "Received unsupported value: " << msg->data << std::endl;
            return;
        }

        *log_file_ << "Function result: " << result << std::endl;
    }

    void handle_string_message(const std_msgs::msg::String::SharedPtr msg)
    {
        // Log del messaggio ricevuto
        *log_file_ << "Received string message of size: " << msg->data.size() << " bytes" << std::endl;
    }

    int function1()
    {
        *log_file_ << "Executing function1" << std::endl;
        return 10; // valore di ritorno di esempio
    }

    int function2()
    {
        *log_file_ << "Executing function2" << std::endl;
        return 20; // valore di ritorno di esempio
    }

    rclcpp::Subscription<std_msgs::msg::Int32>::SharedPtr subscription_int_;
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_string_;
    std::shared_ptr<std::ofstream> log_file_;
};

int main(int argc, char *argv[])
{
    rclcpp::init(argc, argv);

    // Apri il file di log
    auto log_file = std::make_shared<std::ofstream>("/home/fsimoni/ros2_foxy_ws/output.txt", std::ios_base::app);

    auto node = std::make_shared<ServerNode>(log_file);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}

```

Procedendo al benchmarking nel caso di scambio di interi (come avevamo prima) si ottiene:

- Nel caso di scambi di messaggi poco frequente (circa 10 al secondo) un basso utilizzo della CPU pari a 2.8%
<p>&nbsp;</p>
<div align="center">
  <img src="/100ms-int.png" alt="100ms-int.png">
</div>
<p>&nbsp;</p>

- Nel caso di scambi reale (200 al secondo) 14%, circa 4 volte in più
<p>&nbsp;</p>
<div align="center">
  <img src="/cpu-5ms-int.png" alt="cpu-5ms-int.png">
</div>
<p>&nbsp;</p>

_In entrambi i casi_, la RAM occupata è pari a 50MB

<p>&nbsp;</p>
<div align="center">
  <img src="/ram-int.png" alt="ram-int.png">
</div>
<p>&nbsp;</p>



Procedendo invece con scambi di messaggi a 400KB, si ottiene:

- Nel caso di scambi di messaggi poco frequente (circa 10 al secondo) un basso utilizzo della CPU pari a 42.9%
<p>&nbsp;</p>
<div align="center">
  <img src="/cpu-100ms-400kb.png" alt="cpu-100ms-400kb.png">
</div>
<p>&nbsp;</p>

- Nel caso di scambi reale (200 al secondo) 168%, anche qui circa 4 volte in più. Da notare che necessita quindi di un altro core.
<p>&nbsp;</p>
<div align="center">
  <img src="/cpu-5ms-400kb.png" alt="cpu-5ms-400kb.png">
</div>
<p>&nbsp;</p>

_In entrambi i casi_, la RAM occupata è pari a 100MB circa.
<p>&nbsp;</p>
<div align="center">
  <img src="/ram-400kb.png" alt="ram-400kb.png">
</div>
<p>&nbsp;</p>


### Analisi dei consumi a tempo di compilazione

Si procede dunque a capire i consumi in termini di memoria/CPU a tempo di compilazione di questi pacchetti scelti. Per prima cosa, notiamo che la grandezza del progetto è pari a circa 800MB:

<p>&nbsp;</p>
<div align="center">
  <img src="/peso.jpg" alt="peso.jpg">
</div>
<p>&nbsp;</p>

Per analizzare i consumi conviene creare un ambiente completamente nuovo clonando i sorgenti modificati in `src` nella nuova cartella.
Inoltre abbiamo implementato due script:

- `script.sh` : implementa tutta la logica che automatizza il processo di compilazione dei pacchetti spiegato sopra

- `script_monitor.sh`: permette esecuzione e analisi di `script.sh`

In allegato il codice di `script_monitor.sh`:

```bash
#!/bin/bash

# Nome del file dello script da monitorare
SCRIPT_TO_RUN="script.sh"

# File di log
COMPILATION_LOG="compilation_log.txt"
VMSTAT_LOG="vmstat_log.txt"
TOP_LOG="top_log.txt"

# Inizio registrazione dei log
{
    echo "Compilazione iniziata: $(date)"
    /usr/bin/time -v bash $SCRIPT_TO_RUN 2>&1 | tee -a $COMPILATION_LOG
    echo "Compilazione terminata: $(date)"
} 2>&1 | tee $COMPILATION_LOG &

# PID dello script di compilazione
COMPILATION_PID=$!

# Inizio monitoraggio delle prestazioni
vmstat 1 > $VMSTAT_LOG &
VMSTAT_PID=$!

top -b -d 1 -p $COMPILATION_PID > $TOP_LOG &
TOP_PID=$!

# Attendere la fine della compilazione
wait $COMPILATION_PID

# Fermare il monitoraggio delle prestazioni
kill $VMSTAT_PID
kill $TOP_PID

echo "Monitoraggio completato. I log sono stati salvati in $COMPILATION_LOG, $VMSTAT_LOG e $TOP_LOG."

```

In particolare ci siamo soffermati su come monitorare dettagliatamente le risorse utilizzate durante la compilazione di ROS. 

Lo script avvia la registrazione dei log di compilazione, utilizzando `/usr/bin/time` per tracciare il tempo di esecuzione e le risorse consumate. Parallelamente, avvia `vmstat` e `top` per monitorare rispettivamente l'uso della memoria e della CPU in tempo reale. Questi dati vengono salvati nei file di log `compilation_log.txt`, `vmstat_log.txt` e `top_log.txt`.

Attraverso i dati emersi abbiamo prodotto i seguenti grafici:


<p>&nbsp;</p>
<div align="center">
  <img src="/DiagrammaTortaConsumi.jpeg" alt="DiagrammaTortaConsumi.jpeg">
</div>
<p>&nbsp;</p>

Il diagramma a torta per l'uso della CPU mostra una predominanza dell'attività 'utente' (compilazione di ROS), che rappresenta il 70% dell'utilizzo totale della CPU. Le attività di sistema costituiscono il 9,4%, mentre il 20,4% della CPU rimane inattivo. Da notare il 'wait usage cpu' (tempo di attesa della cpu) molto basso ~ 0.2%, questo implica un'ottima efficenza della CPU. Il termine "wait" (attesa) si riferisce al tempo durante il quale la CPU è inattiva in attesa che vengano completate operazioni di I/O (Input/Output). In pratica, la CPU non sta eseguendo istruzioni poiché sta aspettando che dispositivi esterni completino le loro operazioni di lettura o scrittura. 

Per quanto riguarda la memoria, il grafico evidenzia che il 92,9% della memoria è libera, con solo il 3,9% usato e il 3,2% destinato a buffer/cache. Abbiamo reputato interessante studiare il comportamento della CPU rispetto al tempo. 

<p>&nbsp;</p>
<div align="center">
  <img src="/GraficoCPU.jpeg" alt="GraficoCPU.jpeg">
</div>
<p>&nbsp;</p>

Ci teniamo a precisare che il grafico sovrastante è una registrazione dei primi ~ 15 minuti per motivi dimostrativi. 

Il tempo zero (x=0) è caratterizzato dall' avvio di colcon ossia l'inizio del processo di compilazione. Inoltre abbiamo reputato sensato quello di sommare 'User CPU' + 'System CPU' in modo da valutare complessiavamente il risultato finale della macchina sotto stress dall'operazione di compilazione. Ovviamente avendo visto i risultati dal grafico a torta avevamo delle aspettative che sono state rispettate.

Il processo di compilazione avvinee con 4 CORE (limitazione hardware). Ogni core si predisponde di compilare un pacchetto. Nel momento in cui un pacchetto è compilato viene rilasciata quell'area di memoria e viene associato un ulteriore pacchetto. I picchi del grafico rilevano proprio gli istanti in cui un pacchetto viene rilasciato e successivaente un secondo viene associato al CORE. Ad ogni modo a regime di compilazione viene usato il 100% di CPU

Per una panoramica completa dei valori ottenuti abbiamo sintetizzato il tutto attraverso questa taballa§

<p>&nbsp;</p>
<div align="center">
  <img src="/StatisticheOutput.png" alt="StatisticheOutput.png">
</div>
<p>&nbsp;</p>



## Conclusioni

### Obiettivo di porting soddisfatto

Durante questo progetto abbiamo approfondito e navigato attraverso la complessa esperienza di porting di ROS2 su un'architettura RISC-V, scoprendo sfide tecniche e cercando varie soluzioni possibili. Durante questo percorso, siamo riusciti a superare diversi ostacoli, adattando ROS a un ambiente non supportato, dimostrandone la possibilità di realizzazione.

Dopo aver quindi spiegato il funzionamento di ROS e dopo un'analisi del modo in cui viene utilizzato su Ubuntu, ci siamo immersi su RISC-V provando la compilazione di un pacchetto per la creazione di nodi. L'esito è andato a buon fine e siamo riusciti a creare nodi di esempio completamente funzionanti. Abbiamo quindi dimostrato che è possibile creare nodi comunicanti tra loro tramite topic (e non solo) ed esposto benchmark e analisi completa a tempo di compilazione.

La nostra esperienza sottolinea l'importanza di una solida comprensione delle basi di ROS 2 e delle pratiche di low-level-programming per navigare con successo nel suo ecosistema, specialmente quando si affrontano sfide legate a hardware e architetture specifiche.

Speriamo quindi che questo documento non solo serva come tutorial su come installare ROS su RISC-V, ma anche a come approcciarsi ad un qualsiasi porting su questo tipo di architettura. Di fatti qui si possono notare tutte le strategie che possono essere applicate, come l'evitare porzioni non essenziali durante l'installazione, la modifica ad hoc del compilatore, la risoluzione step-by-step per ogni dipendenza, e così via.

### Sviluppi futuri

Il progetto mette quindi in mostra qualche esempio di esecuzione di nodi. In futuro, si possono ovviamente aumentare le funzionalità di questi ultimi aggiungendo ad esempio altre librerie rendendolo quindi utilizzabile in un vero e proprio sistema hardware automatico come robot o comunque collegandosi a sensori fisici su boards.

Inoltre, si potrebbero applicare questi passaggi per installare altre librerie come `rclpy`, equivalente di `rclcpp` ma per nodi in python. Così facendo, si potrebbero creare nodi ben più complessi e moderni per un controllo molto più esteso su vari componenti Hardware.

Infine, un'altro sviluppo potrebbe essere quello di concentrarsi sul funzionamento dei pacchetti di test, ignorati in questa guida per motivi di tempo. Il loro funzionamento permetterebbe di testare in modo specifico i nodi creati.




