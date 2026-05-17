async function gerarProva() {
    // Pegando os valores digitados pelo professor
    const apiKey = document.getElementById('apiKey').value;
    const nomeEscola = document.getElementById('nomeEscola').value;
    const nomeProfessor = document.getElementById('nomeProfessor').value;
    const assunto = document.getElementById('assunto').value;
    const qtdQuestoes = document.getElementById('qtdQuestoes').value;
    const pesoQuestao = document.getElementById('pesoQuestao').value;
    const tipoQuestao = document.getElementById('tipoQuestao').value;
    const conteudoBase = document.getElementById('conteudoBase').value;
    
    const resultadoDiv = document.getElementById('resultado');
    const btnPrint = document.getElementById('btnPrint');

    // Validação simples
    if (!apiKey || !nomeEscola || !nomeProfessor || !assunto || !conteudoBase) {
        alert("Por favor, preencha todos os campos e insira sua API Key do Gemini.");
        return;
    }

    resultadoDiv.style.display = "block";
    resultadoDiv.innerText = "Gerando prova... Por favor, aguarde.";

    // Engenharia de Prompt baseada nas escolhas do professor
    let regraFormato = "";
    if (tipoQuestao === "objetiva") {
        regraFormato = `Crie questões de múltipla escolha. Cada questão deve ter exatamente 5 alternativas (a, b, c, d, e). Adicione um gabarito simples no final da prova.`;
    } else {
        regraFormato = `Crie questões dissertativas (abertas), deixando linhas em branco ou espaço indicado para a resposta do aluno. Adicione critérios de correção sugeridos no final para o professor.`;
    }

    const prompt = `
Você é um assistente acadêmico especialista em criar avaliações escolares.
Crie uma prova com base nas seguintes especificações e no conteúdo de apoio fornecido.

=== CABEÇALHO DA PROVA ===
Escola: ${nomeEscola}
Professor(a): ${nomeProfessor}
Matéria/Assunto: ${assunto}
Aluno(a): ________________________________________ Data: ____/____/______

=== REGRAS DAS QUESTÕES ===
- Quantidade de questões: ${qtdQuestoes}
- Valor de cada questão: ${pesoQuestao} pontos
- Formato: ${regraFormato}

=== CONTEÚDO DE APOIO (Use apenas este texto para criar as questões) ===
${conteudoBase}

Por favor, formate o texto final pronto para impressão, com o cabeçalho no topo, seguido pelas questões numeradas e pontuadas.
`;

    // Chamada oficial para a API do Gemini 3 Flash
    try {
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                contents: [{
                    parts: [{ text: prompt }]
                }]
            })
        });

        const data = await response.json();
        
        if (data.candidates && data.candidates[0].content.parts[0].text) {
            const provaGerada = data.candidates[0].content.parts[0].text;
            resultadoDiv.innerText = provaGerada;
            btnPrint.style.display = "block"; // Mostra o botão de imprimir
        } else {
            resultadoDiv.innerText = "Erro ao formatar a resposta da IA. Verifique os dados.";
        }

    } catch (error) {
        console.error("Erro na requisição:", error);
        resultadoDiv.innerText = "Erro ao conectar com a API. Verifique sua chave de acesso e conexão.";
    }
}

// Função para abrir a janela de impressão com o texto da prova
function imprimirProva() {
    const conteudoProva = document.getElementById('resultado').innerText;
    const janelaImpressao = window.open('', '', 'height=600,width=800');
    
    janelaImpressao.document.write('<html><head><title>Imprimir Prova</title>');
    janelaImpressao.document.write('<style>body { font-family: "Times New Roman", serif; padding: 40px; line-height: 1.6; white-space: pre-wrap; }</style>');
    janelaImpressao.document.write('</head><body>');
    janelaImpressao.document.write(conteudoProva);
    janelaImpressao.document.write('</body></html>');
    
    janelaImpressao.document.close();
    janelaImpressao.print();
}
