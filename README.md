# LSP---retorna-parametros-de-horario
Função que retorna parâmetros de um determinado horário.

As configurações do Sênior não tratam os horários de 24 horas (plantões) corretamente, logo não há funções internas para utilizarmos nestas situações

## Código
```
@==============================================================================@

Definir Funcao retornaCargaHorariaETurno(Numero nCodHorario);

Definir Numero nCargaHorariaDiaria;     @ Variável de saída. @
Definir Numero nMetadeCargaHorariaDiaria; @ Variável de saída. @
Definir Numero nEhHorarioNoturno;  @ Quando igual a 1 indica que o horário é noturno. Quando igual a 0 indica que o horário é diurno. @
Definir Numero nEhHorario24h;  @ Quando igual a 1 indica que o horário é de 24h - plantão. @
Definir Numero nToleranciaSaidaAntecipada;  @ Variável principal @
Definir Numero nToleranciaAtraso;  @ Variável principal @
Definir Numero nHoraEntradaToleranciaAntes; @ Variável principal - define se a marcação é valida ou não, pois marcações muito fora do horário não devem ser consideradas. @
Definir Numero nHoraSaidaToleranciaApos;  @ Variável principal - define se a marcação é valida ou não, pois marcações muito fora do horário não devem ser consideradas. @
Definir Numero nHoraMetadeDoPeriodo; @ Variável principal @

Funcao retornaCargaHorariaETurno(Numero nCodHorario);
{ 
     @ --------------------------- Cálculo da carga horária diária  -------------------------- @
     @ --------------------------- Recupera valores de tolerâncias  -------------------------- @
     @ ----------------------- De acordo com as configurações do horario  -------------------- @
     @                                                                                         @ 
     @                   Hora entrada                            Hora saída                    @
     @  Tempo > -----*---X----*------------------------------*---X----*------ >                @
     @               |         entrada tolerância DEPOIS     |         saída tolerância DEPOIS @
     @               entrada tolerância ANTES                saída tolerância ANTES            @
     @                                                                                         @

     Definir Cursor cCargaHorariaDiaria;
     Definir Numero nCargaHorariaAlmoco; nCargaHorariaAlmoco = 0;
     Definir Numero nSequenciaMarcacao; nSequenciaMarcacao = 0; 

     @ Inicializa valores. @
     nCargaHorariaDiaria = 0;
     nMetadeCargaHorariaDiaria = 0;
     nEhHorarioNoturno = 0;
     nEhHorario24h = 0;
     nToleranciaSaidaAntecipada = 0; 
     nToleranciaAtraso = 0;          
     nHoraEntradaToleranciaAntes = 0;
     nHoraSaidaToleranciaApos = 0;
     nHoraMetadeDoPeriodo = 0;
     
     nHoraSaidaToleranciaAnterior = 0;     

     nHoraEntrada = 0; @ variáveis de apoio @
     nHoraSaida = 0; @ variáveis de apoio @
     nPrimeiraBatida = 0; @ variáveis de apoio @
     nSegundaBatida = 0; @ variáveis de apoio @
     nTerceiraBatida = 0; @ variáveis de apoio @
     nQuartaBatida = 0; @ variáveis de apoio @

     cCargaHorariaDiaria.SQL "select R004MHR.SEQMAR, R004MHR.HORBAT, R004MHR.TOLANT, R004MHR.TOLAPO from R004MHR where R004MHR.CODHOR = :nCodHorario";
     
     cCargaHorariaDiaria.abrirCursor(); 
     Enquanto (cCargaHorariaDiaria.achou){ 
        Se(cCargaHorariaDiaria.SEQMAR = 1){
              nPrimeiraBatida = cCargaHorariaDiaria.HORBAT; 
              nHoraEntrada = cCargaHorariaDiaria.HORBAT; 
              nHoraEntradaToleranciaAntes = cCargaHorariaDiaria.TOLANT;
              nToleranciaAtraso = cCargaHorariaDiaria.TOLAPO - cCargaHorariaDiaria.HORBAT; 
        }
        Se(cCargaHorariaDiaria.SEQMAR = 2){
              nSegundaBatida = cCargaHorariaDiaria.HORBAT; 
              nHoraSaida = cCargaHorariaDiaria.HORBAT; 
              nHoraSaidaToleranciaApos = cCargaHorariaDiaria.TOLAPO;
              nToleranciaSaidaAntecipada =  cCargaHorariaDiaria.HORBAT - cCargaHorariaDiaria.TOLANT;  
              nHoraSaidaToleranciaAnterior = cCargaHorariaDiaria.TOLANT;
              nSequenciaMarcacao = cCargaHorariaDiaria.SEQMAR;  
        }
        Se(cCargaHorariaDiaria.SEQMAR = 3){nTerceiraBatida = cCargaHorariaDiaria.HORBAT;}
        Se(cCargaHorariaDiaria.SEQMAR = 4){
              nQuartaBatida = cCargaHorariaDiaria.HORBAT; 
              nHoraSaida = cCargaHorariaDiaria.HORBAT; 
              nHoraSaidaToleranciaApos = cCargaHorariaDiaria.TOLAPO;
              nToleranciaSaidaAntecipada =  cCargaHorariaDiaria.HORBAT - cCargaHorariaDiaria.TOLANT;  
              nHoraSaidaToleranciaAnterior = cCargaHorariaDiaria.TOLANT;
              nSequenciaMarcacao = cCargaHorariaDiaria.SEQMAR;             
              
        }
        cCargaHorariaDiaria.proximo();
     }    
     cCargaHorariaDiaria.fecharCursor();  
     
     @ Se o horário possue 04 batidas @
     Se(nCodHorario < 9996)
     inicio
         Se(nQuartaBatida <> 0)
         inicio
             nCargaHorariaAlmoco = nTerceiraBatida - nSegundaBatida;
             nCargaHorariaDiaria = nQuartaBatida - nPrimeiraBatida - nCargaHorariaAlmoco;     
         fim;
         Senao @ Se o horário possue 02 batidas, então não tem almoço e apenas pega a diferença da hora da saída e da entrada para ter a carga horario total. @ 
         inicio
             nCargaHorariaDiaria = nSegundaBatida - nPrimeiraBatida;
             Se((nPrimeiraBatida > nSegundaBatida)) {
                nEhHorarioNoturno = 1;
             }
             Se(nPrimeiraBatida = nSegundaBatida) {   @ ATENÇÃO -> O Sênior não trata os horários de 24 horas. @
                nEhHorario24h = 1;
                nCargaHorariaDiaria = 24*60;
                nEhHorarioNoturno = 2; @ Seta um valor inválido. @       
             }
             Se(nEhHorarioNoturno = 1){ @ Horário noturno. Por exemplo: 19:00 - 07:00. @
                 nCargaHorariaDiaria = (24*60 - nPrimeiraBatida) + nSegundaBatida;
             }
         fim;  
     fim;
     nMetadeCargaHorariaDiaria = nCargaHorariaDiaria/2;

     @ É necessário saber se a batida é de entrada ou saída. Para isto, @       
     @ devemos encontrar qual horas corresponde à metade da carga horária diária. @
     @ Para os horários noturnos e de 24h, a metade é definida pela virada do 1º para o 2º dia.@     
     nHoraMetadeDoPeriodo = nHoraEntrada + ((nHoraSaida-nHoraEntrada)/2);  @ Para ajudar na definição se a batida é na primeira metade ou na segunda metade do horário. @
}
```

## Recomendações
Para usar a função na regra, deve-se declará-la no início da mesma.
É interessante que se reserve, por exemplo, a regra 001 apenas para implementar funções.

Em implementações de relatório, tenho o hábito de declarar as funções em Funções Globais no Modelo Gerador.

![image](https://github.com/heripedroso/LSP---converte-minutos-em-HH-MI/assets/22459829/fa6ef8f7-399d-4923-9c2e-a814f502bddc)
