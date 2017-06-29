## network.conf and int DescribeNetwork()

The network.conf file decribes all neural populations, receptors, and connections.  Similar to the network.pro file, there is 1 statement per line.

The file is organized as a list of neural populations.  Each population declaration begins with
     NeuralPopulation: [name]
and ends with
     EndNeuralPopulation

Within each population declaration is a list of population parameters, followed by a list of receptor declarations, followed by a list of target declarations.

The available population parameters are listed below.  Several population parameters take on specific values by default, so not all parameters must be listed in the file.

     N=              .Ncells : number of neurons in the population

     C=              .C : capacitance (nF)
     Taum=           .Taum : membrane time constant
     RestPot=        .RestPot : cell resting potential (mV)
     ResetPot=       .ResetPot : cell reset potential after spike
     Threshold=      .Threshold : Threshold for emitting a spike (mV)

     RestPot_ca=     .Vk : Resting potential for calcium-activated potassium channels (mV)
     Alpha_ca=       .alpha_ca : Amount of increment of [Ca] with each spike discharge. (muM)
     Tau_ca=         .tau_ca : Time constant for calcium-activated potassium channels (msec)
     Eff_ca=         .g_ahp : Efficacy for calcium-activated potassium channels (nS)

         //Anomalous delayed rectifier (ADR)
     g_ADR_Max=      .g_adr_max (= 0) : Maximun value of the g
     V_ADR_h=        .Vadr_h (= -100) : Potential for g_adr=0.5g_adr_max
     V_ADR_s=        .Vadr_s (= 10) : Slope of g_adr at Vadr_h, defining sharpness of g_ard shape.
     ADRRevPot=      .ADRRevPot (= -90) : Reverse potential for ADR
     g_k_Max=        .g_k_max (= 0) : Maximum outward rectifying current
     V_k_h=          .Vk_h (= -34) : Potential for g_k=0.5g_k_max
     V_k_s=          .Vk_s (= 6.5) : Slope of g_k at Vk_h, defining how sharp the shape of g_k is.
     tau_k_max=      .tau_k_max (= 8) : maximum tau for outward rectifying k current

     tauhm=          .tauhm (= 20) : ?
     tauhp=          .tauhp (= 100) : ?
     V_h=            .V_h (= -60) : ?
     V_T=            .V_T (= 120) : ?
     g_T=            .g_T (= 0.0) : ?

After these population parameters, and still within the population declaration, is a list of receptor declarations. Each receptor declaration begins with
     Receptor: [name]
and ends with
     EndReceptor

Within each receptor declaration is a list of receptor parameters.  The available receptor parameters are listed below.

     Tau=            .Tau[currentreceptor] : tau for conductance, see EQ 5/6
     RevPot=         .RevPots[...] : Reversal potential
     FreqExt=        .FreqExt[...] : External frequency
     FreqExtSD=      .FreqExtSD[...] (= 0) : External frequency standard deviation
     MeanExtEff=     .MeanExtEff[...] : External efficacy
     ExtEffSD=       .ExtEffSD[...] (= 0) : External efficacy standard deviation
     MeanExtCon=     .MeanExtCon[...] : External connections

There is also a .MgFlag[...] parameter, corresponding to magnesium block, which is set to true if the receptor name is NMDA.

Following the receptor declarations, and still within the population declaration, is a list of declarations for each target population (representing synaptic projections from population to target population).  Each target population declaration starts with
     TargetPopulation: [name]
and ends with
     EndTargetPopulation

Within each target declaration is a list of parameters.

     Connectivity=          .Connectivity : mean fraction of randomly connected target neurons
     TargetReceptor=        .TargetReceptor : Target receptor code (name)
     MeanEff=               .MeanEfficacy : mean efficacy (for initialization)
     EffSD=                 .EfficacySD (= 0) : Standard deviation of the efficacy distribution.
     STDepressionP=         .pv (= 0) : Short-term depression P (see equation 7)
     STDepressionTau=       .tauD (= 1000) : Short-term depression Tau (see equation 7)
     STFacilitationP=       .Fp (= 0) : Short-term facilitation P (see equation 7)
     STFacilitationTau=     .tauF (= 5000) :  Short-term facilitation Tau (see equation 7)

The function parses the file in two passes, the first to gather the names of populations and receptors, and the second for all of the details.


     // =================================================================
     // Descriptors to generate the network structure

     typedef struct {
       float Connectivity; // mean fraction of randomly connected target neurons
       float TargetReceptor; // 0=AMPA, ...
       float MeanEfficacy; // mean efficacy (for initialization)
       float EfficacySD; // Standard deviation of the efficacy distribution.
       float pv;  // Each spike reduce the fraction D of available vesicle by the factor pv.
       float tauD; // The time constant for the speed of D recovery.
       float Fp;   // Each spike increase F by the factor Fp.
       float tauF; // The time constant for the speed of F decrease.
     } SynPopDescr;

     typedef struct {
       char Label[100];
       int Ncells;

       int NTargetPops;
       int TargetPops[MAXP];

       SynPopDescr SynP[MAXP];

       int Nreceptors;
       char  ReceptorLabel[MAXRECEPTORS][100]; // label for the receptor (needed for the compiler)
       float Tau[MAXRECEPTORS]; // tau for each conductance
       float RevPots[MAXRECEPTORS]; // reversal potentials
       int   MgFlag[MAXRECEPTORS]; // magnesium block flag

       // external input (externally defined)
       float MeanExtCon[MAXRECEPTORS]; // mean total number of external connections
       float MeanExtEff[MAXRECEPTORS]; // external mean efficacy
       float ExtEffSD[MAXRECEPTORS]; // Deviation of distribution of external efficacy
       float FreqExt[MAXRECEPTORS]; // external frequency in Hz
       float FreqExtSD[MAXRECEPTORS]; // external frequency in Hz
       float FreqExtMem[MAXRECEPTORS]; // memory in the external noise
       float FreqExtDecay[MAXRECEPTORS]; //decay factor of the external noise
       float FreqExtNorm[MAXRECEPTORS]; //Normalization factor for the memory of the external noise

       // external input (statistics internally calculated)
       float MeanExtMuS[MAXRECEPTORS]; // statistics of the external input nS/s
       float MeanExtSigmaS[MAXRECEPTORS];
       // dynamic variables

       float C; // capacitance
       float Taum; // membrane time constant
       float RestPot; // Resting potential
       float ResetPot; // reset potential
       float Threshold; // threhsold

       float Vk;  // resting potential
       float alpha_ca; // Amount of increment of [Ca] with each spike discharge. (muM)
       float tau_ca; // time constant
       float g_ahp; // efficacy

       //Anomalous delayed rectifier (ADR)
       float g_adr_max;  //Maximun value of the g
       float Vadr_h; //Potential for g_adr=0.5g_adr_max
       float Vadr_s; //Slop of g_adr at Vadr_h, defining how sharp the shape of g_ard is.
       float ADRRevPot; //Reverse potential for ADR
       float g_k_max;  // Maximun outward rectifying current
       float Vk_h; //Potential for g_k=0.5g_k_max
       float Vk_s; //Slop of g_k at Vk_h, defining how sharp the shape of g_k is.
       float tau_k_max; //maximum tau for outward rectifying k current

       float tauhm;
       float tauhp;
       float V_T;
       float V_h;
       float g_T;
       float h;
     } PopDescr;

     PopDescr PopD[MAXP];


     // =======================================================================
     // DescribeNetwork()
     // Initializes the descriptors of the network by parsing cl_network.conf
     // =======================================================================

     #define EXC 0
     #define INH 1


     int DescribeNetwork() {
       FILE* devconf;
       char buf[1000], *s, *es;
       int currentpopflag = 0;
       int currentreceptorflag = 0;
       int line, auxi;
       int currentpop, currentreceptor, currenttarget;
       float aux;
       float std_p, std_tau;
       int flag_d;

       // FIRST PASSAGE
       // -------------------------------------------------------------------
       // the parser has to go over the file twice: the first time reads all
       // the labels and initializes the number of populations and the number
       // of receptors
       report("Parsing network configuration... first passage\n");

       /*  strncpy=(network_conf,prefix,strlen(prefix));
       strcpy=(network_conf+strlen(prefix),"network.conf");
       devconf=fopen(network_conf,"r");*/
       devconf = fopen("network.conf", "r");
       if (devconf == NULL) {
         printf("ERROR:  Unable to read configuration file\n");
         return 0;
       }

       Npop = 0;
       line = -1;

       while (fgets(buf, 1000, devconf) != NULL) {
         line++;
         s = buf;
         // trim \n at the end
         es = buf;
         while (*es && *es != '\n') es++;
         *es = 0;

         while (*s == ' ' || *s == '\t') s++;  // skip blanks
         if (*s == 0) continue;                // empty line
         if (*s == '%') continue;              // remark

         // commands for defining a new population
         if (strncmp(s, "NeuralPopulation:", 17) == 0) {
           currentpopflag = 1;
           s += 17;
           while (*s == ' ') s++;
           strcpy(PopD[Npop].Label, s);
           report("Population: %s\n", PopD[Npop].Label);
           PopD[Npop].Nreceptors = 0;
           continue;
         }

         if (strncmp(s, "EndNeuralPopulation", 19) == 0) {
           currentpopflag = 0;
           Npop++;
           continue;
         }

         // command for defining a receptor
         if (strncmp(s, "Receptor:", 9) == 0) {
           currentreceptorflag = 1;
           s += 9;
           while (*s == ' ') s++;
           strcpy(PopD[Npop].ReceptorLabel[PopD[Npop].Nreceptors], s);
           report("Receptor %d: %s\n", PopD[Npop].Nreceptors,
                  PopD[Npop].ReceptorLabel[PopD[Npop].Nreceptors]);
         }

         if (strncmp(s, "EndReceptor", 11) == 0) {
           currentreceptorflag = 0;
           PopD[Npop].Nreceptors++;
         }
       }

       fclose(devconf);

       // Second passage: now all the parameters and the target populations are
       // parsed
       // ----------------------------------------------------------------------------

       report("Parsing network configuration... second passage\n");

       //  devconf=fopen(network_conf,"r");
       devconf = fopen("network.conf", "r");
       if (devconf == NULL) {
         printf("ERROR:  Unable to read configuration file\n");
         return 0;
       }

       line = -1;

       while (fgets(buf, 1000, devconf) != NULL) {
         line++;
         s = buf;

         // trim \n at the end
         es = buf;
         while (*es && *es != '\n') es++;
         *es = 0;

         while (*s == ' ' || *s == '\t') s++;  // skip blanks
         if (*s == 0) continue;                // empty line
         if (*s == '%') continue;              // remark

         // population commands
         if (strncmp(s, "NeuralPopulation:", 17) == 0) {
           currentpopflag = 1;
           s += 17;
           while (*s == ' ') s++;
           currentpop = PopulationCode(s);
           if (currentpop == -1) {
             printf("Unknown population [%s]: line %d\n", s, line);
             return -1;
           }
           PopD[currentpop].NTargetPops = 0;
           report(
               "-----------------------------------\n    Population: "
               "%s\n-----------------------------------\n",
               PopD[currentpop].Label);

           // Initialize some population parameters

           PopD[currentpop].g_adr_max = 0;
           PopD[currentpop].Vadr_h = -100;
           PopD[currentpop].Vadr_s = 10;
           PopD[currentpop].ADRRevPot = -90;
           PopD[currentpop].g_k_max = 0;
           PopD[currentpop].Vk_h = -34;
           PopD[currentpop].Vk_s = 6.5;
           PopD[currentpop].tau_k_max = 8;

           PopD[currentpop].tauhm = 20;
           PopD[currentpop].tauhp = 100;
           PopD[currentpop].V_h = -60;
           PopD[currentpop].V_T = 120;
           PopD[currentpop].g_T = 0.0;
           continue;
         }

         if (strncmp(s, "EndNeuralPopulation", 19) == 0) {
           report("EndPopulation\n");
           currentpopflag = 0;
           continue;
         }

         // paramters for the population

         if (strncmp(s, "N=", 2) == 0 && currentpopflag) {
           PopD[currentpop].Ncells = atoi(s + 2);
           report("  N=%d\n", PopD[currentpop].Ncells);
           continue;
         }
         if (strncmp(s, "C=", 2) == 0 && currentpopflag) {
           PopD[currentpop].C = atof(s + 2);
           report("  C=%f nF\n", (double)PopD[currentpop].C);
           continue;
         }
         if (strncmp(s, "Taum=", 5) == 0 && currentpopflag) {
           PopD[currentpop].Taum = atof(s + 5);
           report("  Membrane time constant=%f ms\n", (double)PopD[currentpop].Taum);
           continue;
         }
         if (strncmp(s, "RestPot=", 8) == 0 && currentpopflag) {
           PopD[currentpop].RestPot = atof(s + 8);
           report("  Resting potential=%f mV\n", (double)PopD[currentpop].RestPot);
           continue;
         }
         if (strncmp(s, "ResetPot=", 9) == 0 && currentpopflag) {
           PopD[currentpop].ResetPot = atof(s + 9);
           report("  Reset potential=%f mV\n", (double)PopD[currentpop].ResetPot);
           continue;
         }
         if (strncmp(s, "Threshold=", 10) == 0 && currentpopflag) {
           PopD[currentpop].Threshold = atof(s + 10);
           report("  Threshold =%f mV\n", (double)PopD[currentpop].Threshold);
           continue;
         }

         if (strncmp(s, "RestPot_ca=", 11) == 0 && currentpopflag) {
           PopD[currentpop].Vk = atof(s + 11);
           report("  RestPot_ca =%f mV\n", (double)PopD[currentpop].Vk);
           continue;
         }

         if (strncmp(s, "Alpha_ca=", 9) == 0 && currentpopflag) {
           PopD[currentpop].alpha_ca = atof(s + 9);
           report("  Alpha_ca =%f mV\n", (double)PopD[currentpop].alpha_ca);
           continue;
         }

         if (strncmp(s, "Tau_ca=", 7) == 0 && currentpopflag) {
           PopD[currentpop].tau_ca = atof(s + 7);
           report("  Tau_ca =%f mV\n", (double)PopD[currentpop].tau_ca);
           continue;
         }

         if (strncmp(s, "Eff_ca=", 7) == 0 && currentpopflag) {
           PopD[currentpop].g_ahp = atof(s + 7);
           report("  Eff_ca =%f mV\n", (double)PopD[currentpop].g_ahp);
           continue;
         }

         if (strncmp(s, "g_ADR_Max=", 10) == 0 && currentpopflag) {
           PopD[currentpop].g_adr_max = atof(s + 10);
           report("  g_adr_max=%f mV\n", (double)PopD[currentpop].g_adr_max);
           continue;
         }

         if (strncmp(s, "V_ADR_h=", 8) == 0 && currentpopflag) {
           PopD[currentpop].Vadr_h = atof(s + 8);
           report("  V_ADR_h=%f mV\n", (double)PopD[currentpop].Vadr_h);
           continue;
         }

         if (strncmp(s, "V_ADR_s=", 8) == 0 && currentpopflag) {
           PopD[currentpop].Vadr_s = atof(s + 8);
           report("  V_ADR_s=%f mV\n", (double)PopD[currentpop].Vadr_h);
           continue;
         }

         if (strncmp(s, "ADRRevPot=", 10) == 0 && currentpopflag) {
           PopD[currentpop].ADRRevPot = atof(s + 10);
           report("  ADRRevPot=%f mV\n", (double)PopD[currentpop].ADRRevPot);
           continue;
         }

         if (strncmp(s, "g_k_Max=", 8) == 0 && currentpopflag) {
           PopD[currentpop].g_k_max = atof(s + 8);
           report("  g_k_max=%f mV\n", (double)PopD[currentpop].g_k_max);
           continue;
         }

         if (strncmp(s, "V_k_h=", 6) == 0 && currentpopflag) {
           PopD[currentpop].Vk_h = atof(s + 6);
           report("  V_k_h=%f mV\n", (double)PopD[currentpop].Vk_h);
           continue;
         }

         if (strncmp(s, "V_k_s=", 6) == 0 && currentpopflag) {
           PopD[currentpop].Vk_s = atof(s + 6);
           report("  V_k_s=%f mV\n", (double)PopD[currentpop].Vk_h);
           continue;
         }
         if (strncmp(s, "tau_k_max=", 10) == 0 && currentpopflag) {
           PopD[currentpop].tau_k_max = atof(s + 10);
           report("  tau_k_max=%f mV\n", (double)PopD[currentpop].tau_k_max);
           continue;
         }

         if (strncmp(s, "tauhm=", 6) == 0 && currentpopflag) {
           PopD[currentpop].tauhm = atof(s + 6);
           report("  tauhm=%f ms\n", (double)PopD[currentpop].tauhm);
           continue;
         }
         if (strncmp(s, "tauhp=", 6) == 0 && currentpopflag) {
           PopD[currentpop].tauhp = atof(s + 6);
           report("  tauhp=%f ms\n", (double)PopD[currentpop].tauhp);
           continue;
         }
         if (strncmp(s, "V_h=", 4) == 0 && currentpopflag) {
           PopD[currentpop].V_h = atof(s + 4);
           report("  V_h=%f mV\n", (double)PopD[currentpop].V_h);
           continue;
         }
         if (strncmp(s, "V_T=", 4) == 0 && currentpopflag) {
           PopD[currentpop].V_T = atof(s + 4);
           report("  V_T=%f mV\n", (double)PopD[currentpop].V_T);
           continue;
         }
         if (strncmp(s, "g_T=", 4) == 0 && currentpopflag) {
           PopD[currentpop].g_T = atof(s + 4);
           report("  g_T=%f mS\n", (double)PopD[currentpop].g_T);
           continue;
         }
         // receptor paramters
         if (strncmp(s, "Receptor:", 9) == 0 && currentpopflag) {
           currentreceptorflag = 1;
           s += 9;
           while (*s == ' ') s++;
           currentreceptor = ReceptorCode(s, currentpop);
           if (currentreceptor == -1) {
             printf("ERROR: Unknown receptor type\n");
             return -1;
           }
           if (strncmp(s, "NMDA", 4) ==
               0) {  // activate magnesium block for NMDA type
             PopD[currentpop].MgFlag[currentreceptor] = 1;
           } else
             PopD[currentpop].MgFlag[currentreceptor] = 0;
           report("Receptor %d: %s (Mg block: %d)\n", currentreceptor,
                  PopD[currentpop].ReceptorLabel[currentreceptor],
                  PopD[currentpop].MgFlag[currentreceptor]);

           PopD[currentpop].ExtEffSD[currentreceptor] = 0;
           PopD[currentpop].FreqExtSD[currentreceptor] = 0;
           continue;
         }

         if (strncmp(s, "EndReceptor", 11) == 0) {
           currentreceptorflag = 0;
           continue;
         }

         if (strncmp(s, "Tau=", 4) == 0 && currentpopflag && currentreceptorflag) {
           PopD[currentpop].Tau[currentreceptor] = atof(s + 4);
           report("  Tau=%f ms\n", (double)PopD[currentpop].Tau[currentreceptor]);
           continue;
         }
         if (strncmp(s, "RevPot=", 7) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].RevPots[currentreceptor] = atof(s + 7);
           report("  Reversal potential=%f mV\n",
                  (double)PopD[currentpop].RevPots[currentreceptor]);
           continue;
         }
         if (strncmp(s, "FreqExt=", 8) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].FreqExt[currentreceptor] = atof(s + 8);
           report("  Ext frequency=%f Hz\n",
                  (double)PopD[currentpop].FreqExt[currentreceptor]);
           continue;
         }
         if (strncmp(s, "FreqExtSD=", 10) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].FreqExtSD[currentreceptor] = atof(s + 10);
           report("  Ext frequency SD=%f Hz\n",
                  (double)PopD[currentpop].FreqExtSD[currentreceptor]);
           continue;
         }
         if (strncmp(s, "MeanExtEff=", 11) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].MeanExtEff[currentreceptor] = atof(s + 11);
           report("  Ext efficacy=%f nS\n",
                  (double)PopD[currentpop].MeanExtEff[currentreceptor]);
           continue;
         }
         if (strncmp(s, "ExtEffSD=", 9) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].ExtEffSD[currentreceptor] = atof(s + 9);
           report("  Ext efficacy SD=%f nS\n",
                  (double)PopD[currentpop].ExtEffSD[currentreceptor]);
           continue;
         }
         if (strncmp(s, "MeanExtCon=", 11) == 0 && currentpopflag &&
             currentreceptorflag) {
           PopD[currentpop].MeanExtCon[currentreceptor] = atof(s + 11);
           report("  Ext connections=%f\n",
                  (double)PopD[currentpop].MeanExtCon[currentreceptor]);
           continue;
         }

         // target populations

         if (strncmp(s, "TargetPopulation:", 17) == 0 && currentpopflag) {
           s += 17;
           while (*s == ' ') s++;
           currenttarget = PopulationCode(s);
           if (currenttarget == -1) {
             printf("Unknown target population: line %d\n", line);
             return -1;
           }
           PopD[currentpop].TargetPops[PopD[currentpop].NTargetPops] = currenttarget;

           report("Synapses targeting population: %s (%d)\n ",
                  PopD[currenttarget].Label, currenttarget);

           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].pv = 0;
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].tauD = 1000;
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].Fp = 0;
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].tauF = 5000;
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].EfficacySD = 0;
           continue;
         }

         if (strncmp(s, "EndTargetPopulation", 20) == 0) {
           PopD[currentpop].NTargetPops++;
           continue;
         }

         if (strncmp(s, "Connectivity=", 13) == 0 && currentpopflag) {
           aux = atof(s + 13);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].Connectivity = aux;
           report("  Connectivity=%f\n", (double)aux);
         }
         if (strncmp(s, "TargetReceptor=", 15) == 0 && currentpopflag) {
           s += 15;
           while (*s == ' ' || *s == '\t') s++;
           auxi = ReceptorCode(s, currenttarget);
           if (auxi == -1) {
             printf("Unknown target receptor, line %d\n", line);
             return -1;
           }
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].TargetReceptor = auxi;
           report("  Target receptor code=%d\n", auxi);
         }

         if (strncmp(s, "MeanEff=", 8) == 0 && currentpopflag) {
           aux = atof(s + 8);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].MeanEfficacy = aux;
           report("  Mean efficacy=%f\n", (double)aux);
         }
         if (strncmp(s, "EffSD=", 6) == 0 && currentpopflag) {
           aux = atof(s + 6);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].EfficacySD = aux;
           report("  Efficacy SD=%f\n", (double)aux);
         }

         if (strncmp(s, "STDepressionP=", 14) == 0 && currentpopflag) {
           aux = atof(s + 14);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].pv = aux;
           report("  STDepressionP=%f mV\n", (double)aux);
         }

         if (strncmp(s, "STDepressionTau=", 16) == 0 && currentpopflag) {
           aux = atof(s + 16);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].tauD = aux;
           report("  STDepressionTau=%f mV\n", (double)aux);
         }
         if (strncmp(s, "STFacilitationP=", 16) == 0 && currentpopflag) {
           aux = atof(s + 16);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].Fp = aux;
           report("  STFacilitationP=%f mV\n", (double)aux);
         }

         if (strncmp(s, "STFacilitationTau=", 18) == 0 && currentpopflag) {
           aux = atof(s + 18);
           PopD[currentpop].SynP[PopD[currentpop].NTargetPops].tauF = aux;
           report("  STFacilitationTau=%f mV\n", (double)aux);
         }
       }  // end while

       fclose(devconf);

       return 1;
     }