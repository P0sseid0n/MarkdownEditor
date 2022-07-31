<script setup lang="ts">
import { reactive, nextTick } from 'vue';

const blocks = reactive([
   '# Title 1',
   '## Title 2',
   '### Title 3',
])


function getBlockStyled(value: string): string {
   const text = {
      titleSize: 0,
      valueEscaped: value
   }

   if (value.startsWith('######')) text.titleSize = 6
   else if (value.startsWith('#####')) text.titleSize = 5
   else if (value.startsWith('####')) text.titleSize = 4
   else if (value.startsWith('###')) text.titleSize = 3
   else if (value.startsWith('##')) text.titleSize = 2
   else if (value.startsWith('#')) text.titleSize = 1

   text.valueEscaped = text.valueEscaped.substring(text.titleSize).replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#39;")

   if (text.titleSize > 0) {
      return `<h${text.titleSize}>${text.valueEscaped}</h${text.titleSize}>`
   } else {
      return `<p>${text.valueEscaped}</p>`
   }
}

function inputDivText(payload: Event) {
   const pre = payload.target as HTMLDivElement
   blocks.length = 0
      ;[...pre.children].forEach((element, index) => {
         const p = element as HTMLParagraphElement
         blocks[index] = p.innerText
      })
}
</script>

<template>
   <section id="input">
      <main>
         <pre contenteditable @input="inputDivText"
            @blur="inputDivText"><p v-for="block in blocks" v-text="block" v-once ></p></pre>
      </main>
   </section>
   <section id="output">
      <main>
         <span v-for="(block, index) in blocks" v-html="getBlockStyled(block)"></span>
      </main>
   </section>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;700&display=swap');

* {
   font-family: 'Poppins', sans-serif;
   margin: 0;
   padding: 0;
   box-sizing: border-box;
}

#app {
   display: flex;
   flex-direction: row;
   height: 100vh;
   padding: 0 8px;

   >section {
      width: 50%;
      height: 100%;
      padding: 32px 16px;

      >main {
         width: 100%;
         height: 100%;
      }
   }
}

#input {
   border-right: 4px solid rgb(225, 225, 225);

   pre {
      width: 100%;
      font-size: 16px;
      border: none;
      outline: none;
      line-height: 32px;
   }
}

#output {
   border-left: 4px solid rgb(225, 225, 225);

   h1 {
      font-size: 36px;
      font-weight: 700;
      margin-bottom: 32px;
   }

   h2 {
      font-size: 32px;
      font-weight: 700;
      margin-bottom: 24px;
   }

   h3 {
      font-size: 28px;
      font-weight: 700;
      margin-bottom: 16px;
   }

   h4 {
      font-size: 24px;
      font-weight: 700;
      margin-bottom: 8px;
   }

   h5 {
      font-size: 20px;
      font-weight: 700;
      margin-bottom: 4px;
   }

   h6 {
      font-size: 16px;
      font-weight: 700;
   }

   p {
      padding: 0 8px;
   }

   h1,
   h2,
   h3 {
      border-bottom: 1px solid rgb(205, 205, 205);
   }


}
</style>