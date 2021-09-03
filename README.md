# Utilizando Blockchain com JavaScript

## 00: Como funciona o Blockchain

- O Blockchain é um protocolo de registro distribuído. Cada bloco que faz parte da cadeia (de blocos) é protegido por um código criptografado e armazena uma informação. Quando o bloco é válidado e se junta aos demais blocos ele ganha um registro permanente que não pode ser alterado. Isso significa que sempre que um bloco já válidado é alterado toda a cadeia se torna inválida, fazendo com que o Blockchain seja uma maneira segura de executar transações.

## 01: Criando a Blockchain

- O primeiro passo é definirmos como um bloco na nossa Blockchain deverá ser. Criamos a classe Block dentro do arquivo ```main.js``` e dentro dela adicionamos um construtor que recebe como parâmetro as propriedades de um bloco. Adicionamos também o hash de um bloco que será calculado por uma função.

```
class Block{
    constructor(index, timestamp, data, previousHash = ''){
        this.index = index;
        this.timestamp = timestamp;
        this.data = data;
        this.previousHash = previousHash;
        this.hash = '';
    }
}
```

- Agora que o construtor já está criado, vamos criar uma função para calcular o nosso hash. Nesta função iremos utilizar o SHA-256 para calcular o hash e como o JavaScript não possuí esta criptografia precisaremos importar uma biblioteca rodando o comando a seguir

```
npm install --save crypto-js
```

- Não esqueça de importar a biblioteca no início do seu arquivo

```
const SHA256 = require('crypto-js/sha256');
```

- Agora que já importamos nossa biblioteca podemos implementar o cálculo do nosso hash. Dentro do método criado iremos retornar o SHA256 dos dados definidos no construtor em formato de string. Seu método deverá se parecer com isso:

```
calculateHash(){
    return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data)).toString();
}
```

- Não esqueça de modificar o construtor para que o hash do nosso bloco seja calculado com a função que criamos.

```
constructor(index, timestamp, data, previousHash = ''){
    this.index = index;
    this.timestamp = timestamp;
    this.data = data;
    this.previousHash = previousHash;
    this.hash = this.calculateHash();
}
```

- Agora vamos criar nossa classe Blockchain e dentro dela definir o construtor que será a responsável por inicializar nossa Blockchain. Nosso construtor terá uma propriedade chamada ```chain``` que nada mais é do que um array de blocos. 

- O primeiro bloco da nossa blockchain é denomidado ```Genesis Block``` e ele deve ser definido manualmente. Faremos isso através de uma função ```createGenesisBlock()```. A função irá retornar um novo bloco.

- Depois de criada a função basta chamá-la dentro do array de chain para que seja definido nosso Genesis Block.

```
class Blockchain{
    constructor(){
        this.chain = [this.createGenesisBlock()];
    }

    createGenesisBlock(){
        return new Block(0, "01/01/2021", "Genesis Block", "0");
    }
}
```

- Vamos também criar alguns métodos que podem ser úteis no futuro. Primeiro um método para recuperar o último bloco.

```
getLatestBlock(){
    returh this.chain[this.chain.lenght -1];
}
```

- Agora vamos implementar o método para adicionar um novo bloco a nossa cadeia. Para isso precisamos primeiro atribuir a hash do bloco antigo à propriedade ```previousHash``` do novo bloco. Depois precisamos calcular o hash desse novo bloco. Por fim iremos adicioná-lo a nossa cadeia.

```
addBlock(newBlock){
    newBlock.previousHash = this.getLatestBlock().hash;
    newBlock.hash = newBlock.calculateHash();
    this.chain.push(newBlock);
}
```

- Vamos testar se nosso código funciona. Adicione as seguintes linhas no final do seu arquivo

```
let isnotcoin = new Blockchain();
isnotcoin.addBlock(new Block(1, "02/11/2021", { amount: 4 }));
isnotcoin.addBlock(new Block(2, "04/11/2021", { amount: 10 }));

console.log(JSON.stringify(isnotcoin, null, 4));
```

- Se você executar o arquivo com ```node main.js``` será impresso no seu terminal nossa cadeia contendo nossos blocos. Note que os blocos estão interligados por suas hashs, onde a previousHash de um bloco corresponde a hash do bloco anterior ao dele. 

- Como sabemos, se alterarmos um bloco na nossa cadeia nós iremos invalidar a cadeia inteira. No nosso caso nós não estamos verificando a integridade da nossa cadeia, estamos apenas criando-a. Para isso vamos adicionar um novo método na nossa classe ```Blockchain```.

- No nosso método iremos adicionar um loop que passará por toda nossa cadeia, lembrando que o loop deve ter início 1 pois nosso primeiro bloco é o Genesis. Dentro do nosso loop iremos recuperar o bloco atual e o bloco anterior para podermos dar início aos testes. 

    1º - Verificamos se a hash do bloco atual é igual a hash calculada. 

    2º - Verificamos se o atributo previousHash do bloco atual é igual ao hash do bloco anterior.

- Retornamos falso em ambos os casos e, caso percorra todo o loop sem cair em um dos if's, retornamos true informando que a cadeia é válida.

```
isChainValid(){
    for(let i = 1; i < this.chain.length; i++){
        const currentBlock = this.chain[i];
        const previousBlock = this.chain[i -1];

        if(currentBlock.hash !== currentBlock.calculateHash()){
            return false;
        }

        if(currentBlock.previousHash !== previousBlock.hash){
            return false;
        }
    }

    return true;
}
```

- Com isso verificamos a integridade da nossa cadeia, ou seja, caso algum bloco seja alterado a cadeia se torna inválida. Mesmo que você altere o hash de um bloco a cadeia será inválidada pois o relacionamento deste bloco com o anterior foi quebrado.

## 02: Proof of Work

- Até este ponto nós já conseguimos criar blocos e validar nossa cadeia fácilmente, o problema é que caso alguém queira spammar nossa cadeia e adicionar múltiplos blocos por segundo será possível e a proof of work, ou prova de trabalho, nos ajuda a evitar este spam.

- Proof of Work nada mais é do que um algoritmo que é demorado e muito caro de se produzir mas é fácil de se verificar que é válido. No Bitcoin é conhecido como "minerar", um processo demorado e custoso mas que, quando finalizado, é de fácil validação. No caso do Bitcoin, ele exige que o hash comece com uma certa quantidade de zeros, e como não podemos influenciar o cálculo de um hash o que nos resta é tentar inúmeras combinações até que achemos uma que bate com um hash cálculado.

- Vamos ver isso na prática. Dentro da nossa classe ```Block``` iremos adicionar uma função chamada ```mineBlock``` e fazer com que o hash de um bloco comece sempre com uma quantidade de zeros.

- Dentro da função iremos colocar um loop while. Iremos pegar uma substring da hash começando com 0 e que irá até o valor passado como parâmetro para a função. O while deverá rodar até que a substring da hash seja igual quantidade de zeros necessárias.

```
mineBlock(difficulty){
    while(this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")){
        this.hash = this.calculateHash();
    }

    console.log("Block mined! Hash: " + this.hash);
}
```

- Aqui temos um pequeno problema pois o hash do nosso bloco não irá mudar a menos que o conteúdo do mesmo seja alterado, ou seja, nosso while é um loop infinito. Como não podemos alterar o index, timestamp e os outros valores do nosso bloco precisamos de outra solução. 

- O Blockchain possuí um conceito chamado de ```Nonce (Number Only Used Once)```, que nada mais é do que um número aleatório que não tem nada a ver com o bloco e que pode ser alterado. O Nonce é o que os mineradores buscam, uma vez que a solução é achada eles recebem seu bloco.

- Adicione a propriedade nonce no construtor da classe ```Block``` e iremos inicializar com 0. Depois iremos incrementá-la no nosso loop e também adicionar o nonce no cálculo da nossa hash. Sua classe block deve ficar assim:

```
class Block{
    constructor(index, timestamp, data, previousHash = ''){
        this.index = index;
        this.timestamp = timestamp;
        this.data = data;
        this.previousHash = previousHash;
        this.hash = this.calculateHash();
        this.nonce = 0;
    }

    calculateHash(){
        return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data) + this.nonce).toString();
    }

    mineBlock(difficulty){
        while(this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")){
            this.nonce++;
            this.hash = this.calculateHash();
        }

        console.log("Block mined! Hash: " + this.hash);
    }
}
```

- Agora vamos adicionar o sistema que criamos a nossa Blockchain. Para isso iremos alterar a função ```addBlock```, agora ao invés de cálcularmos o hash diretamente iremos utilizar o nosso ```mineBlock``` passando a dificuldade como parâmetro. Como a dificuldade poderá ser utilizada novamente depois podemos também adicioná-la ao construtor da classe ```Blockchain```. Sua classe ficará assim: 

```
class Blockchain{
    constructor(){
        this.chain = [this.createGenesisBlock()];
        this.difficulty = 2;
    }

    createGenesisBlock(){
        return new Block(0, "01/01/2021", "Genesis Block", "0");
    }

    getLatestBlock(){
        return this.chain[this.chain.length -1];
    }

    addBlock(newBlock){
        newBlock.previousHash = this.getLatestBlock().hash;
        newBlock.mineBlock(this.difficulty);
        this.chain.push(newBlock);
    }

    isChainValid(){
        for(let i = 1; i < this.chain.length; i++){
            const currentBlock = this.chain[i];
            const previousBlock = this.chain[i -1];

            if(currentBlock.hash !== currentBlock.calculateHash()){
                return false;
            }

            if(currentBlock.previousHash !== previousBlock.hash){
                return false;
            }
        }

        return true;
    }
}
```

- Se rodarmos nosso arquivo no terminal iremos notar que o bloco é minerado de forma extremamente rápida. Isso pode ser evitado incrementendado a dificuldade para se minerar um bloco. Com este mecânismo nós controlamos o quão rápido um bloco pode ser adicionado na nossa cadeia.
