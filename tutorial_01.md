# Tutorial: Criando um App de Lista de Tarefas com Next.js

Neste tutorial, você aprenderá como criar um Web App de Lista de Tarefas usando Next.js com TypeScript e Tailwind CSS. O objetivo é criar uma lista de tarefas simples, com funcionalidades para adicionar e remover tarefas. Futuramente, você poderá integrar um banco de dados e fazer deploy, mas essa parte será abordada em outra etapa.

## 1. Configuração do Projeto
### Passo 1: Criar o Projeto Next.js com TypeScript
No terminal CMD do Windows, navegue até a pasta (diretório) onde você salva seus projetos e execute o seguinte comando para criar o projeto:

```bash 
npx create-next-app@latest to-do-app
```

```bash
√ Would you like to use TypeScript? ... No / Yes
√ Would you like to use ESLint? ... No / Yes
√ Would you like to use Tailwind CSS? ... No / Yes
√ Would you like to use `src/` directory? ... No / Yes
√ Would you like to use App Router? (recommended) ... No / Yes
√ Would you like to customize the default import alias (@/*)? ... No / Yes
```

Esse comando vai criar uma pasta (diretório) com o nome que você criou seu projeto, no caso *to-do-app*

Entre na pasta do projeto com o comando abaixo:

```bash
cd to-do-app
```

### Passo 2: Instalar Tailwind CSS

Instale o Tailwind CSS e suas dependências:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```
Obs: Você precisa estar na pasta do projeto antes de fazer a instalação acima, pois essa instalação é local no projeto. Para cada projeto que você for utilizar o tailwind é preciso executar os comandos acima.

Abra o projeto no editor de código VSCode:

```bash
code .
```

Configure o arquivo tailwind.config.js com o conteúdo abaixo:

```ts
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/**/*.{js,ts,jsx,tsx}', // Adiciona o tailwind nos arquivos do diretório /src com extesão .js .ts .jsx .tsx
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Agora, edite o arquivo *./src/app/globals.css* e adicione as seguintes linhas:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Essa configuração importa o tailwind para o CSS global da nossa aplicação.

### Passo 3: Limpar o Projeto
Remova o conteúdo padrão do arquivo *./src/app/page.tsx*, pois vamos começar do zero.

## 2. Construção do App de Lista de Tarefas
### Passo 4: Criar a Estrutura da Página
No arquivo *src/app/page.tsx*, crie a estrutura do nosso aplicativo com o código abaixo:

```tsx
"use client";

import React, { useState } from "react";

type Task = {
  id: number;
  text: string;
  dueDate: string;
};

export default function Home() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [newTask, setNewTask] = useState("");
  const [dueDate, setDueDate] = useState("");

  const addTask = () => {
    if (!newTask || !dueDate) return;

    const newTaskItem: Task = {
      id: Date.now(),
      text: newTask,
      dueDate: dueDate,
    };

    setTasks([...tasks, newTaskItem]);
    setNewTask("");
    setDueDate("");
  };

  const deleteTask = (id: number) => {
    setTasks(tasks.filter((task) => task.id !== id));
  };

  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center">
      <div className="bg-gray-800 p-8 rounded shadow-lg w-full max-w-md">
        <h1 className="text-2xl font-bold mb-4 text-center text-gray-100">Lista de Tarefas</h1>
        <div className="mb-4">
          <input
            type="text"
            placeholder="Descrição da tarefa"
            value={newTask}
            onChange={(e) => setNewTask(e.target.value)}
            className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
          />
          <input
            type="datetime-local"
            value={dueDate}
            onChange={(e) => setDueDate(e.target.value)}
            className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
          />
          <button
            onClick={addTask}
            className="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700 transition"
          >
            Adicionar Tarefa
          </button>
        </div>

        <ul className="space-y-2">
          {tasks.map((task) => (
            <li
              key={task.id}
              className="flex justify-between items-center p-2 bg-gray-700 rounded"
            >
              <div>
                <p className="text-gray-200">{task.text}</p>
                <p className="text-sm text-gray-500">{task.dueDate}</p>
              </div>
              <button
                onClick={() => deleteTask(task.id)}
                className="bg-red-500 text-white p-1 rounded hover:bg-red-600 transition"
              >
                Excluir
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

## 3. Executar o App
### Passo 5: Executar o App
Para rodar o servidor de desenvolvimento, use o comando:

```bash
npm run dev
```

Agora, acesse *http://localhost:3000* no navegador para visualizar seu app de Lista de Tarefas.

## 4. Explicação do Código:

### Clint-side:

```bash
"use client"
```
O código usa a diretiva "use client", que instrui o Next.js a renderizar essa página no lado do cliente, permitindo o uso de *hooks* como *useState*. Como estamos utilizando o App Router do Next.js, essa diretiva é necessária para marcar o arquivo como um componente client-side (ao contrário do server-side rendering padrão do Next.js).

### Importação do React:

```bash
import React, { useState } from "react";
```

O React é importado, e o useState é utilizado para gerenciar o estado da aplicação no lado do cliente.

### Tipagem:

```ts
type Task = {
  id: number;
  text: string;
  dueDate: string;
};
```
Define-se um tipo Task que representa uma tarefa com três propriedades:
#### **id:** número que serve como identificador único da tarefa (baseado no timestamp da data de criação).
#### **text:** a descrição da tarefa.
#### **dueDate:** uma string que representa a data e hora de vencimento da tarefa.

Essa definição em TypeScript está criando um tipo chamado Task. Um tipo em TypeScript define a estrutura esperada para um objeto, especificando quais propriedades ele deve ter e quais tipos de valores essas propriedades devem armazenar.

### Componente Principal (Home):

```ts
export default function Home() {
```
A função Home é o componente principal exportado da página. Ela contém a lógica e a interface para gerenciar a lista de tarefas.

### Estados (useState):

```ts
const [tasks, setTasks] = useState<Task[]>([]);
const [newTask, setNewTask] = useState("");
const [dueDate, setDueDate] = useState("");
```
#### **tasks:** armazena a lista de tarefas. Começa como um array vazio e é atualizado à medida que as tarefas são adicionadas ou removidas.
#### **newTask:** armazena o texto da nova tarefa a ser adicionada. Inicialmente vazio.
#### **dueDate:** armazena a data de vencimento da nova tarefa

### Função de Adicionar Tarefa (addTask):

```ts
const addTask = () => {
  if (!newTask || !dueDate) return;

  const newTaskItem: Task = {
    id: Date.now(),
    text: newTask,
    dueDate: dueDate,
  };

  setTasks([...tasks, newTaskItem]);
  setNewTask("");
  setDueDate("");
};
```
A função addTask cria uma nova tarefa e a adiciona à lista tasks.
Verifica se tanto newTask quanto dueDate estão preenchidos. Se algum deles estiver vazio, a função simplesmente retorna, impedindo a adição.
Cria um objeto newTaskItem com um id único (baseado no timestamp atual Date.now()), o texto da tarefa (newTask) e a data de vencimento (dueDate).
Atualiza o estado tasks usando o spread operator [...] para adicionar a nova tarefa ao final da lista.
Limpa os campos newTask e dueDate para que o usuário possa adicionar uma nova tarefa.

### Função de Excluir Tarefa (deleteTask):

```ts
const deleteTask = (id: number) => {
  setTasks(tasks.filter((task) => task.id !== id));
};
```
A função deleteTask remove uma tarefa com base em seu id.
A função filter cria uma nova lista que exclui a tarefa cujo id foi passado.
O estado tasks é atualizado com essa nova lista sem a tarefa removida.

### Estrutura da Página:

```ts
return (
  <div className="min-h-screen bg-gray-900 flex items-center justify-center">
    <div className="bg-gray-800 p-8 rounded shadow-lg w-full max-w-md">
      <h1 className="text-2xl font-bold mb-4 text-center text-gray-100">Lista de Tarefas</h1>
      <div className="mb-4">
        <input
          type="text"
          placeholder="Descrição da tarefa"
          value={newTask}
          onChange={(e) => setNewTask(e.target.value)}
          className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
        />
        <input
          type="datetime-local"
          value={dueDate}
          onChange={(e) => setDueDate(e.target.value)}
          className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
        />
        <button
          onClick={addTask}
          className="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700 transition"
        >
          Adicionar Tarefa
        </button>
      </div>

      <ul className="space-y-2">
        {tasks.map((task) => (
          <li
            key={task.id}
            className="flex justify-between items-center p-2 bg-gray-700 rounded"
          >
            <div>
              <p className="text-gray-200">{task.text}</p>
              <p className="text-sm text-gray-500">{task.dueDate}</p>
            </div>
            <button
              onClick={() => deleteTask(task.id)}
              className="bg-red-500 text-white p-1 rounded hover:bg-red-600 transition"
            >
              Excluir
            </button>
          </li>
        ))}
      </ul>
    </div>
  </div>
);
```

Um div principal define uma tela de altura mínima (min-h-screen), com o fundo escuro (bg-gray-900) e usa o Flexbox para centralizar o conteúdo.
O div interno (bg-gray-800) contém padding, bordas arredondadas e uma sombra, criando o "card" que contém o conteúdo.
O título "Lista de Tarefas" é estilizado e centralizado.
Inputs para Adicionar Tarefa:

Um campo de entrada de texto (input[type="text"]) que armazena a descrição da nova tarefa, com valor vinculado ao estado newTask e um evento onChange para atualizar o estado.
Um campo de data e hora (input[type="datetime-local"]) para a data de vencimento, vinculado ao estado dueDate.
Um botão "Adicionar Tarefa" que chama a função addTask.
Lista de Tarefas:

Um ul é gerado dinamicamente usando tasks.map() para iterar sobre as tarefas.
Cada tarefa é renderizada como um item de lista (li) com a descrição (task.text) e data de vencimento (task.dueDate).
Um botão "Excluir" que remove a tarefa correspondente ao clicar, chamando a função deleteTask.

### Tailwind CSS:

O código utiliza o Tailwind CSS para estilização. Algumas classes-chave são:
bg-gray-900, bg-gray-800, bg-gray-700: diferentes tonalidades de fundo cinza.
p-8, p-2: padding.
rounded: bordas arredondadas.
hover:bg-blue-700, hover:bg-red-600: transições ao passar o mouse sobre os botões.
text-gray-100, text-gray-300, text-gray-500: diferentes tons de texto.
w-full: largura total dos inputs e botões.
space-y-2: espaçamento vertical entre os itens da lista