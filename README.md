# AEP2S_PSE

Integrantes:

ANDREI SIPIANI ROMERO RA: 26001554-2 

MATHEUS FELIPE DE SOUZA ZAMFERRARI RA: 26011299-2

#include <stdio.h>
#include <locale.h>

struct tp_acao {
    char codigo[8];
    char descricao[100];
    char data[20];
    char escola[50];
    char tema[50];
    char publico[50];
    char responsavel[50];
    char situacaoInicial[20];
    int quantP;
    int quantEfetiva;
};

tp_acao acao[100];
int ultimo = 0;

int tamanhoTexto(const char texto[]) {
    int i = 0;

    while (texto[i] != '\0') {
        i++;
    }

    return i;
}

bool lerLinha(char destino[], int tamanho) {
    int comprimento;
    int caractere;

    if (fgets(destino, tamanho, stdin) == NULL) {
        destino[0] = '\0';
        return false;
    }

    comprimento = tamanhoTexto(destino);

    if (comprimento > 0 && destino[comprimento - 1] == '\n') {
        destino[comprimento - 1] = '\0';
    } else if (!feof(stdin)) {
        caractere = getchar();

        if (caractere != '\n' && caractere != EOF) {
            while (caractere != '\n' && caractere != EOF) {
                caractere = getchar();
            }

            destino[0] = '\0';
            return false;
        }
    }

    return true;
}

bool ehEspaco(char caractere) {
    return caractere == ' ' || caractere == '\t' ||
           caractere == '\r' || caractere == '\n';
}

void removerEspacos(char texto[]) {
    int inicio = 0;
    int fim = tamanhoTexto(texto) - 1;
    int i;

    while (texto[inicio] != '\0' && ehEspaco(texto[inicio])) {
        inicio++;
    }

    while (fim >= inicio && ehEspaco(texto[fim])) {
        fim--;
    }

    for (i = 0; inicio <= fim; i++, inicio++) {
        texto[i] = texto[inicio];
    }

    texto[i] = '\0';
}

void copiarTexto(char destino[], const char origem[]) {
    int i = 0;

    while (origem[i] != '\0') {
        destino[i] = origem[i];
        i++;
    }

    destino[i] = '\0';
}

char minuscula(char caractere) {
    if (caractere >= 'A' && caractere <= 'Z') {
        return caractere + ('a' - 'A');
    }

    return caractere;
}

bool textoIgual(const char texto1[], const char texto2[]) {
    int i = 0;

    while (texto1[i] != '\0' && texto2[i] != '\0') {
        if (minuscula(texto1[i]) != minuscula(texto2[i])) {
            return false;
        }

        i++;
    }

    return texto1[i] == '\0' && texto2[i] == '\0';
}

bool contemTexto(const char texto[], const char pesquisa[]) {
    int i;
    int j;

    if (pesquisa[0] == '\0') {
        return false;
    }

    for (i = 0; texto[i] != '\0'; i++) {
        j = 0;

        while (pesquisa[j] != '\0' && texto[i + j] != '\0' &&
               minuscula(texto[i + j]) == minuscula(pesquisa[j])) {
            j++;
        }

        if (pesquisa[j] == '\0') {
            return true;
        }
    }

    return false;
}

void lerTextoObrigatorio(const char mensagem[], char destino[], int tamanho) {
    bool valido;

    do {
        printf("%s", mensagem);
        valido = lerLinha(destino, tamanho);

        if (!valido) {
            printf("ERRO: TEXTO EXCEDEU O LIMITE DE CARACTERES. TENTE NOVAMENTE.\n");
        } else {
            removerEspacos(destino);

            if (destino[0] == '\0') {
                printf("ERRO: CAMPO OBRIGATÓRIO. INFORME UM VALOR.\n");
            }
        }
    } while (!valido || destino[0] == '\0');
}

bool converterInteiro(const char texto[], int *valor) {
    int i = 0;
    int sinal = 1;
    long long numero = 0;

    while (texto[i] == ' ' || texto[i] == '\t') {
        i++;
    }

    if (texto[i] == '-') {
        sinal = -1;
        i++;
    } else if (texto[i] == '+') {
        i++;
    }

    if (texto[i] < '0' || texto[i] > '9') {
        return false;
    }

    while (texto[i] >= '0' && texto[i] <= '9') {
        numero = numero * 10 + (texto[i] - '0');

        if (numero > 2147483647LL) {
            return false;
        }

        i++;
    }

    while (texto[i] == ' ' || texto[i] == '\t') {
        i++;
    }

    if (texto[i] != '\0') {
        return false;
    }

    numero = numero * sinal;

    if (numero < 0 || numero > 2147483647LL) {
        return false;
    }

    *valor = (int)numero;
    return true;
}

int lerQuantidadeNaoNegativa(const char mensagem[]) {
    char entrada[50];
    int valor;
    bool valido;

    do {
        printf("%s", mensagem);
        valido = lerLinha(entrada, 50);

        if (!valido || !converterInteiro(entrada, &valor)) {
            printf("ERRO: VALOR INVÁLIDO. INFORME UM NÚMERO INTEIRO NÃO NEGATIVO.\n");
            valido = false;
        }
    } while (!valido);

    return valor;
}

bool codigoJaCadastrado(const char codigo[]) {
    int i;

    for (i = 0; i < ultimo; i++) {
        if (textoIgual(acao[i].codigo, codigo)) {
            return true;
        }
    }

    return false;
}

void lerCodigoNovo(char codigo[]) {
    do {
        lerTextoObrigatorio("INFORME O CÓDIGO DA AÇÃO: ", codigo, 8);

        if (codigoJaCadastrado(codigo)) {
            printf("ERRO: CÓDIGO JÁ CADASTRADO. INFORME OUTRO CÓDIGO.\n");
        }
    } while (codigoJaCadastrado(codigo));
}

void escolherSituacao(char situacao[]) {
    char opcao[10];
    bool valido = false;

    do {
        printf("\nSITUAÇÃO DA AÇÃO:\n");
        printf("1 - PLANEJADA\n");
        printf("2 - REALIZADA\n");
        printf("3 - CANCELADA\n");
        printf("ESCOLHA UMA OPÇÃO: ");

        if (!lerLinha(opcao, 10)) {
            printf("ERRO: OPÇÃO INVÁLIDA. ESCOLHA 1, 2 OU 3.\n");
        } else if (textoIgual(opcao, "1")) {
            copiarTexto(situacao, "planejada");
            valido = true;
        } else if (textoIgual(opcao, "2")) {
            copiarTexto(situacao, "realizada");
            valido = true;
        } else if (textoIgual(opcao, "3")) {
            copiarTexto(situacao, "cancelada");
            valido = true;
        } else {
            printf("ERRO: SITUAÇÃO INVÁLIDA. ESCOLHA 1, 2 OU 3.\n");
        }
    } while (!valido);
}

void exibirAcao(int posicao) {
    printf("\n  TEMA: %s\n", acao[posicao].tema);
    printf("  ESCOLA: %s\n", acao[posicao].escola);
    printf("  DATA PREVISTA: %s\n", acao[posicao].data);
    printf("  PÚBLICO-ALVO: %s\n", acao[posicao].publico);
    printf("  RESPONSÁVEL: %s\n", acao[posicao].responsavel);
    printf("  CÓDIGO: %s\n", acao[posicao].codigo);
    printf("  QUANTIDADE PREVISTA: %d\n", acao[posicao].quantP);
    printf("  QUANTIDADE EFETIVA: %d\n", acao[posicao].quantEfetiva);
    printf("  SITUAÇÃO: %s\n", acao[posicao].situacaoInicial);
    printf("  DESCRIÇÃO: %s\n", acao[posicao].descricao);
}

void cadastrarAcao() {
    printf("\n--- CADASTRAR AÇÃO ---\n");

    if (ultimo >= 100) {
        printf("LIMITE DE 100 AÇÕES ATINGIDO. NÃO É POSSÍVEL CADASTRAR OUTRA AÇÃO.\n");
        return;
    }

    lerTextoObrigatorio("\nINFORME O TEMA DA AÇÃO: ", acao[ultimo].tema, 50);
    lerTextoObrigatorio("INFORME A ESCOLA DA AÇÃO: ", acao[ultimo].escola, 50);
    lerTextoObrigatorio("INFORME A DATA PREVISTA DA AÇÃO: ", acao[ultimo].data, 20);
    lerTextoObrigatorio("INFORME O PÚBLICO-ALVO DA AÇÃO: ", acao[ultimo].publico, 50);
    lerTextoObrigatorio("INFORME O RESPONSÁVEL PELA AÇÃO: ", acao[ultimo].responsavel, 50);
    lerCodigoNovo(acao[ultimo].codigo);

    acao[ultimo].quantP = lerQuantidadeNaoNegativa(
        "INFORME A QUANTIDADE PREVISTA DE PARTICIPANTES: ");

    escolherSituacao(acao[ultimo].situacaoInicial);

    lerTextoObrigatorio("INFORME A DESCRIÇÃO DA AÇÃO: ", acao[ultimo].descricao, 100);

    if (textoIgual(acao[ultimo].situacaoInicial, "realizada")) {
        acao[ultimo].quantEfetiva = lerQuantidadeNaoNegativa(
            "INFORME A QUANTIDADE EFETIVA DE PARTICIPANTES: ");
    } else {
        acao[ultimo].quantEfetiva = 0;
    }

    ultimo++;

    printf("\n/// AÇÃO CADASTRADA COM SUCESSO ///\n");
}

void listarAcao() {
    int i;

    printf("\n--- LISTA DE AÇÕES CADASTRADAS ---\n");

    if (ultimo == 0) {
        printf("\nNENHUMA AÇÃO CADASTRADA.\n");
        return;
    }

    for (i = 0; i < ultimo; i++) {
        printf("\n%d - AÇÃO\n", i + 1);
        exibirAcao(i);
        printf("----------------------------------------\n");
    }
}

void pesquisarAcao() {
    char pesquisa[100];
    int i;
    int encontradas = 0;

    printf("\n--- PESQUISAR AÇÕES ---\n");

    if (ultimo == 0) {
        printf("NENHUMA AÇÃO CADASTRADA.\n");
        return;
    }

    lerTextoObrigatorio("INFORME O CÓDIGO, A ESCOLA OU O TEMA DA AÇÃO: ", pesquisa, 100);

    for (i = 0; i < ultimo; i++) {
        if (contemTexto(acao[i].codigo, pesquisa) ||
            contemTexto(acao[i].escola, pesquisa) ||
            contemTexto(acao[i].tema, pesquisa)) {

            encontradas++;

            printf("\nRESULTADO %d:\n", encontradas);
            exibirAcao(i);
            printf("----------------------------------------\n");
        }
    }

    if (encontradas == 0) {
        printf("\nAÇÃO NÃO ENCONTRADA.\n");
    }
}

int localizarAcaoPorCodigo(const char codigo[]) {
    int i;

    for (i = 0; i < ultimo; i++) {
        if (textoIgual(acao[i].codigo, codigo)) {
            return i;
        }
    }

    return -1;
}

void atualizarAcao() {
    char codigo[8];
    char novaSituacao[20];
    int posicao;

    printf("\n--- ATUALIZAR SITUAÇÃO DA AÇÃO ---\n");

    if (ultimo == 0) {
        printf("NENHUMA AÇÃO CADASTRADA.\n");
        return;
    }

    lerTextoObrigatorio("INFORME O CÓDIGO DA AÇÃO: ", codigo, 8);

    posicao = localizarAcaoPorCodigo(codigo);

    if (posicao == -1) {
        printf("REGISTRO NÃO ENCONTRADO. CONFIRA O CÓDIGO INFORMADO.\n");
        return;
    }

    printf("\nAÇÃO LOCALIZADA:\n");
    exibirAcao(posicao);

    escolherSituacao(novaSituacao);

    if (textoIgual(novaSituacao, "realizada")) {
        acao[posicao].quantEfetiva = lerQuantidadeNaoNegativa(
            "INFORME A QUANTIDADE EFETIVA DE PARTICIPANTES: ");
    } else {
        acao[posicao].quantEfetiva = 0;
    }

    copiarTexto(acao[posicao].situacaoInicial, novaSituacao);

    printf("\nSITUAÇÃO DA AÇÃO ATUALIZADA COM SUCESSO.\n");
}

void gerarResumo() {
    int planejadas = 0;
    int realizadas = 0;
    int canceladas = 0;
    long long totalParticipantes = 0;
    long long totalPrevistoRealizadas = 0;
    int i;
    double percentual = 0.0;

    printf("\n--- RESUMO GERAL ---\n");

    if (ultimo == 0) {
        printf("NENHUMA AÇÃO CADASTRADA. NÃO HÁ DADOS PARA O RESUMO.\n");
        return;
    }

    for (i = 0; i < ultimo; i++) {
        if (textoIgual(acao[i].situacaoInicial, "planejada")) {
            planejadas++;
        } else if (textoIgual(acao[i].situacaoInicial, "realizada")) {
            realizadas++;
            totalParticipantes += acao[i].quantEfetiva;
            totalPrevistoRealizadas += acao[i].quantP;
        } else if (textoIgual(acao[i].situacaoInicial, "cancelada")) {
            canceladas++;
        }
    }

    if (totalPrevistoRealizadas > 0) {
        percentual = ((double)totalParticipantes / totalPrevistoRealizadas) * 100.0;
    }

    printf("\nTOTAL DE AÇÕES CADASTRADAS: %d\n", ultimo);
    printf("AÇÕES PLANEJADAS: %d\n", planejadas);
    printf("AÇÕES REALIZADAS: %d\n", realizadas);
    printf("AÇÕES CANCELADAS: %d\n", canceladas);
    printf("TOTAL DE PARTICIPANTES EFETIVOS: %lld\n", totalParticipantes);
    printf("TOTAL PREVISTO NAS AÇÕES REALIZADAS: %lld\n", totalPrevistoRealizadas);
    printf("PERCENTUAL DE PARTICIPAÇÃO: %.2f%%\n", percentual);
}

int main() {
    char entrada[10];
    int opcao;
    bool fechar = false;

    if (setlocale(LC_ALL, "Portuguese") == NULL) {
        setlocale(LC_ALL, "pt_BR.UTF-8");
    }

    do {
        printf("\n========== PSE EM AÇÃO ==========\n");
        printf("1 - CADASTRAR AÇÃO\n");
        printf("2 - LISTAR AÇÕES CADASTRADAS\n");
        printf("3 - PESQUISAR AÇÕES\n");
        printf("4 - ATUALIZAR AÇÃO\n");
        printf("5 - GERAR RESUMO\n");
        printf("6 - SAIR\n");
        printf("\nESCOLHA UMA OPÇÃO: ");

        if (!lerLinha(entrada, 10) || !converterInteiro(entrada, &opcao)) {
            printf("\nERRO: OPÇÃO INVÁLIDA. ESCOLHA UMA OPÇÃO DE 1 A 6.\n");
        } else {
            switch (opcao) {
                case 1:
                    cadastrarAcao();
                    break;

                case 2:
                    listarAcao();
                    break;

                case 3:
                    pesquisarAcao();
                    break;

                case 4:
                    atualizarAcao();
                    break;

                case 5:
                    gerarResumo();
                    break;

                case 6:
                    printf("\nSAINDO DO SISTEMA...\n");
                    fechar = true;
                    break;

                default:
                    printf("\nERRO: OPÇÃO INEXISTENTE. ESCOLHA UMA OPÇÃO DE 1 A 6.\n");
                    break;
            }
        }

    } while (!fechar);

    return 0;
}
