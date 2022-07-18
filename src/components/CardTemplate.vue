<template>
  <div class="card">
    <template v-for="node in nodes">
      <template v-if="node.type === 'ColumnSet'">
        <div class="row" :class="node.style">
          <div class="col" v-for="col in node.columns">
            <component v-for="item in col.items" :is="getComponent(item)"></component>
<!--            todo: can be a recursion ? -->
<!--            todo: can columns contains [ColumnsSet and FactSet] ?-->
          </div>
        </div>
      </template>

      <div class="facts-set" v-else-if="node.type === 'FactSet'">
        <component v-for="facts in node.facts" :is="getFactsComponent(facts)"></component>
      </div>

      <template v-else>
        <component :is="getComponent(node)"></component>
      </template>
    </template>
  </div>
</template>

<script lang="ts">
  import {defineComponent, ref, toRaw} from 'vue'

  let extractVars = (initialStr: string) => {
    let found = initialStr.match(/{{[{]?(.*?)[}]?}}/);

    let vars: Array<string> = [];

    if (found && found.length) {
      found.forEach(str => {
        if (str.indexOf('{{') === -1) {
          let topLevelName = str.split('.')[0];
          vars.push(topLevelName);
        }
      })
    }

    return vars;
  }

  export default defineComponent({
    props: {
      scheme: Object,
      initialData: Object
    },
    setup() {
      let nodes = ref(null);

      return {
        nodes,
      }
    },

    provide() {
      return {...this.initialData}
    },

    methods: {
      getComponent(node: object) {
        switch (node.type) {
          //todo: find solution of "Template compilation error: Error parsing JavaScript expression: Unexpected identifier"
          // case 'FactSet': {
          //   let factsStr = '';
          //
          //   node.facts.forEach(facts => {
          //     factsStr += `<component :is="getFactsComponent(${facts})"></component>`
          //   });
          //
          //   return defineComponent({
          //     template: `<div class="facts-set">
          //                   ${factsStr}
          //               </div>`
          //   })
          // }

          case 'TextBlock': {
            return this.getTextBlockComponent(node.text)
          }

          case 'Image': {
            return this.getImageComponent(node)
          }
        }
      },

      getTextBlockComponent(text: string) {
        let vars = extractVars(text);
        let textBlock = defineComponent({
          inject: vars,
          template: `<p class="text-block">${text}</p>`,
        })
        return textBlock
      },

      getImageComponent(image: object) {
        let vars = extractVars(image.url);

        let imageComponent = defineComponent({
          inject: vars,
          template: `<img :src="photo" alt="${image.altText}"/>`,
        })
        return imageComponent
      },

      getFactsComponent(facts: object) {
        let factsComponent = defineComponent({
          inject: [facts.$data],
          template: `<ul class="facts">
                        <li v-for="item in ${facts.$data}">
                               <span class="text-bold">{{item['${facts.title}']}}</span> :  {{item['${facts.value}']}}
                           </li>
                    </ul>`,
        })
        return factsComponent
      },
    },

    mounted() {
      this.nodes = this.scheme?.body;
    }
  })
</script>

<style scoped>
  .card {
    width: 400px;

    border-radius: 10px;
    border: 1px solid #ddd;
    box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;

    overflow: hidden;
  }

  .row {
    display: flex;
  }

  .row.accent {
    padding: 5px 10px;

    background-color: #c1d9f6;
  }

  .row img {
    margin-right: 15px;
  }

  .row .col {
    display: flex;
    flex-direction: column;
  }

  .facts-set {
    padding: 10px;
  }

  .text-block {
    padding: 5px 10px;
  }

  img {
    width: 40px;
    height: 40px;
  }
</style>