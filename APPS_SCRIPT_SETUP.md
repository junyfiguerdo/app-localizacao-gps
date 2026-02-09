# Configuração do Google Apps Script

## O PROBLEMA

O visualizador não estava mostrando os dados capturados porque o Apps Script não estava configurado corretamente para retornar os dados no formato esperado.

## SOL UÇÃO: CÓDIGO DO APPS SCRIPT

Copie e cole o código abaixo em seu Apps Script (Extensiones > Apps Script na sua planilha):

```javascript
// Substitua SHEET_ID pelo ID da sua planilha
const SHEET_ID = 'SEU_ID_DA_PLANILHA_AQUI';
const SHEET_NAME = 'Respostas';  // Nome da aba onde os dados são salvos

function doPost(e) {
  try {
    // Recebe os dados do POST
    const data = JSON.parse(e.postData.contents);
    console.log('Dados recebidos:', data);
    
    // Abre a planilha
    const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME);
    
    // Obtém a última linha
    const lastRow = sheet.getLastRow();
    
    // Adiciona uma nova linha com os dados
    const newRow = lastRow + 1;
    sheet.getRange(newRow, 1, 1, 11).setValues([[
      data.dataHora,
      data.usuario,
      data.empresa,
      data.local,
      data.nome,
      data.endereco,
      data.cidade,
      data.valor,
      data.observacao,
      data.latitude,
      data.longitude
    ]]);
    
    console.log('Dados salvos na linha:', newRow);
    
    return ContentService
      .createTextOutput(JSON.stringify({ success: true, message: 'Dados salvos com sucesso!' }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    console.error('Erro:', error);
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  try {
    // Verifica se a ação é getData
    if (e.parameter.action === 'getData') {
      return getData();
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({ error: 'Ação não reconhecida' }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    console.error('Erro em doGet:', error);
    return ContentService
      .createTextOutput(JSON.stringify({ error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function getData() {
  try {
    const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME);
    const data = sheet.getRange(2, 1, sheet.getLastRow() - 1, 11).getValues();
    
    // Converte os dados em objetos
    const formattedData = data.map(row => ({
      dataHora: row[0],
      usuario: row[1],
      empresa: row[2],
      local: row[3],
      nome: row[4],
      endereco: row[5],
      cidade: row[6],
      valor: row[7],
      observacao: row[8],
      latitude: parseFloat(row[9]),
      longitude: parseFloat(row[10])
    }));
    
    console.log('Dados retornados:', formattedData);
    
    return ContentService
      .createTextOutput(JSON.stringify(formattedData))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    console.error('Erro ao recuperar dados:', error);
    return ContentService
      .createTextOutput(JSON.stringify({ error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## PASSO A PASSO

1. **Abra sua Planilha Google** com os dados de localização

2. **Obtenha o ID da Planilha**
   - A URL da planilha é: `https://docs.google.com/spreadsheets/d/SEU_ID_AQUI/edit`
   - Copie o `SEU_ID_AQUI`

3. **Abra o Apps Script**
   - Na planilha, clique em Extensões > Apps Script
   - Delete o código padrão
   - Cole o código acima

4. **Configure o ID**
   - Na linha `const SHEET_ID = '...';` cole o ID que copiou
   - Verifique se o nome da aba está correto (padrão: 'Respostas')

5. **Salve o projeto** (Ctrl+S)

6. **Deploy como Web App**
   - Clique no botão ↑ (Deploy) no canto superior
   - Selecione "New deployment"
   - Type: "Web app"
   - Execute como: Sua conta
   - Quem tem acesso: "Qualquer pessoa"
   - Clique "Deploy"

7. **Copie a URL de deployment**
   - A URL aparecerá após o deploy
   - Exemplo: `https://script.google.com/macros/s/AkfycbyiMosFim.../usercontent`

8. **Atualize os arquivos HTML**
   - Abra `index.html`
   - Procure por `const SCRIPT_URL = '...'`
   - Substitua pela URL do seu Apps Script
   - Faça o mesmo em `visualizador.html`

## ESTRUTURA DA PLANILHA

Sua planilha deve ter as seguintes colunas (a partir da linha 1):

| Data/Hora | Usuário | Empresa | Local | Nome | Endereço | Cidade | Valor | Observação | Latitude | Longitude |
|-----------|---------|---------|-------|------|----------|--------|-------|-----------|----------|----------|
| (vazio)   | (vazio) | (vazio) | ...   | ...  | ...      | ...    | ...   | ...       | ...      | ...      |

## TESTANDO

1. Abra `index.html` no navegador
2. Permita acesso ao GPS
3. Clique em "Capturar Agora"
4. Verifique o Console (F12) para ver se os dados foram enviados
5. Abra `visualizador.html`
6. Os dados devem aparecer no mapa

## RESOLVENDO PROBLEMAS

**Problema: "Nenhum dado encontrado"**
- Verifique se o Apps Script foi deployado
- Confirme que a URL do Apps Script está correta
- Abra o Apps Script e verifique os logs (Executar > Visualizar logs de execução)

**Problema: "Erro ao salvar"**
- Verifique se tem permissão de escrita na planilha
- Confirme que o ID da planilha está correto

**Problema: "CORS error"**
- Isso foi corrigido na versão mais recente
- Certifique-se de atualizar os arquivos HTML
