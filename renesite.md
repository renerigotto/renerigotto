<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BAIXAR RECIBO DO CAR & LEITOR DE PDF</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Poppins', sans-serif;
        }
        body {
            background-color: #121212;
            color: #ffffff;
            text-align: center;
            padding-top: 50px;
        }
        h2 {
            font-size: 26px;
            font-weight: 600;
            margin-bottom: 20px;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: #1e1e1e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
        }
        input, button, textarea {
            width: 100%;
            padding: 12px;
            margin-top: 10px;
            border-radius: 5px;
            font-size: 16px;
            border: none;
        }
        input {
            background: #2c2c2c;
            color: #fff;
            border: 1px solid #555;
        }
        button {
            background: #4caf50;
            color: #fff;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover {
            background: #45a049;
        }
        textarea {
            height: 500px; /* Aumentei a altura da caixa de texto */
            background: #2c2c2c;
            color: #fff;
            border: 1px solid #555;
            border-radius: 5px;
            resize: none;
        }
    </style>
</head>
<body>
    <h2>DIGITE O NÚMERO DO RECIBO DO CAR</h2>
    <input type="text" id="campoTexto" placeholder="UF-AAAAAAA-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA">
    <button onclick="abrirLink()">DOWNLOAD DO RECIBO</button>
    <input type="file" id="fileInput" accept="application/pdf">
    <textarea id="output"></textarea>
    <button onclick="copiarTexto()">COPIAR</button>

    <script>
        function abrirLink() {
            var texto = document.getElementById("campoTexto").value;
            var url = "https://car.gov.br/pdf/" + texto + "/baixar/1";
            window.open(url, "_blank");
        }

        function copiarTexto() {
            var textarea = document.getElementById("output");
            textarea.select();
            document.execCommand("copy");
            alert("Informações copiadas!");
        }

        document.getElementById('fileInput').addEventListener('change', async function(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.readAsArrayBuffer(file);
            reader.onload = async function() {
                const pdfData = new Uint8Array(reader.result);
                const pdf = await pdfjsLib.getDocument({ data: pdfData }).promise;
                let extractedText = "";

                for (let i = 1; i <= pdf.numPages; i++) {
                    const page = await pdf.getPage(i);
                    const textContent = await page.getTextContent();
                    extractedText += textContent.items.map(item => item.str).join("\n") + "\n";
                }

                const linhas = extractedText.split("\n");
                const extrairValores = (prefixo) => 
                    linhas.filter(linha => linha.startsWith(prefixo)).map(linha => linha.replace(prefixo, "").trim());
                
                const imoveis = extrairValores("Nome do Imóvel Rural:");
                const nomes = extrairValores("Nome:");
                const cpfs = extrairValores("CPF:");
                const cnpjs = extrairValores("CNPJ:");

                let resultado = [];
                resultado.push(...imoveis);
                resultado.push(...nomes);
                resultado.push(...cpfs.map(cpf => `CPF: ${cpf}`));
                resultado.push(...cnpjs.map(cnpj => `CNPJ: ${cnpj}`));
                
                const startIndex = extractedText.indexOf("Município do Cartório");
                const endIndex = extractedText.lastIndexOf("RECIBO DE INSCRIÇÃO DO IMÓVEL RURAL NO CAR");
                
                if (startIndex !== -1 && endIndex !== -1 && endIndex > startIndex) {
                    resultado.push(extractedText.substring(startIndex, endIndex).replace("Município do Cartório", "MATRÍCULA:").trim());
                }

                document.getElementById('output').value = resultado.join("\n");
            };
        });
    </script>
</body>
</html>
 
