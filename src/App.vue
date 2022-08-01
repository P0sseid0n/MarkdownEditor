<script setup lang="ts">
import { reactive, nextTick } from 'vue';

const blocks = reactive([
   '# Title 1',
   '## Title 2',
   '### Title 3',
   '#### Title 4',
   '##### Title 5',
   'Text **bold**, *italic*, ~~strike~~, __underline__, `code`',
   // All markdown syntax
   '- list item 1',
   '- list item 2',
   '1. list item 1',
   '2. list item 2',
   '> blockquote',
   '[Google link](http://google.com/)',
])

function escapeHtml(text: string): string {
   return text.replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#39;")
}


function getBlockStyled(value: string): string {
   const text = {
      start: value.split(' ')[0],
      escaped: escapeHtml(value.substring(value.split(' ')[0].length)),
      raw: value
   }

   text.escaped = text.escaped.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>')
   text.escaped = text.escaped.replace(/\*(.*?)\*/g, '<i>$1</i>')
   text.escaped = text.escaped.replace(/\_\_(.*?)\_\_/g, '<u>$1</u>')
   text.escaped = text.escaped.replace(/\~\~(.*?)\~\~/g, '<s>$1</s>')
   text.escaped = text.escaped.replace(/\`(.*?)\`/g, '<code>$1</code>')

   // text.escaped = text.escaped.replace(/\n/g, '<br>')
   // text.escaped = text.escaped.replace(/\s/g, '&nbsp;')

   if (text.start.includes('#')) return `<h${text.start.length}>${text.escaped}</h${text.start.length}>`
   else if (text.start === '>') return `<blockquote>${text.escaped}</blockquote>`
   else if (text.start === '-') return `<ul><li>${text.escaped}</li></ul>`
   else if (text.raw.match(/\[(.*?)\]\((.*?)\)/g)) return text.raw.replace(/\[(.*?)\]\((.*?)\)/g, '<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>')
   else return `<p>${escapeHtml(text.start) + '' + text.escaped}</p>`
}

function inputDivText(payload: Event) {
   const pre = payload.target as HTMLDivElement
   blocks.length = 0
   const children = [...pre.children]
   children.forEach((element, index) => {
      const p = element as HTMLParagraphElement
      blocks[index] = p.innerText
   })
}

function pasteText(event: ClipboardEvent) {
   const clipboardText = event.clipboardData?.getData('text/plain')


   if (!clipboardText) return

   const path = (event as ClipboardEvent & { path: HTMLElement[] }).path

   const pre = path.find(element => element.tagName === 'PRE') as HTMLPreElement

   const selection = window.getSelection()
   if (!selection?.rangeCount) return

   selection.deleteFromDocument()

   clipboardText.split(/[\r\n]/gm).forEach(text => {

      const p = document.createElement('p')
      if (text === '') p.innerHTML = '<br>'
      else p.innerText = text
      pre.appendChild(p)
   })

   const children = [...pre.children]

   children.forEach((element, index) => {
      const p = element as HTMLParagraphElement
      blocks[index] = p.innerText
   })
}
</script>

<template>
   <section id="input">
      <main>
         <pre contenteditable @paste.prevent="pasteText" @input="inputDivText"
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
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700;800;900&display=swap');

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
      overflow-x: auto;

      >main {
         width: 100%;
         height: 100%;
      }
   }
}

#input {
   border-right: 4px solid rgb(225, 225, 225);

   p {
      word-wrap: break-word;
   }

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
      font-weight: 600;
      margin-bottom: 32px;
   }

   h2 {
      font-size: 32px;
      font-weight: 600;
      margin-bottom: 24px;
   }

   h3 {
      font-size: 28px;
      font-weight: 600;
      margin-bottom: 16px;
   }

   h4 {
      font-size: 24px;
      font-weight: 600;
      margin-bottom: 8px;
   }

   h5 {
      font-size: 20px;
      font-weight: 600;
      margin-bottom: 4px;
   }

   h6 {
      font-size: 16px;
      font-weight: 600;
   }

   p,
   blockquote {
      padding: 0 8px;

      b {
         font-weight: 700;
      }
   }

   h1,
   h2,
   h3 {
      border-bottom: 1px solid rgb(205, 205, 205);
   }

   b {
      font-weight: 800;
   }

   blockquote {
      margin-bottom: 16px;
      border-left: 4px solid rgba(0, 0, 0, 0.2);
      color: rgba(0, 0, 0, 0.4);
      font-weight: 400;
   }

   ul {
      padding-left: 40px;
   }

   code {
      background: rgba(0, 0, 0, 0.075);
      padding: 0 8px;
      border-radius: 4px;
   }

   a {
      color: rgb(6, 133, 212);
      font-weight: 500;

      &:hover {
         text-decoration: none;
      }
   }
}
</style>