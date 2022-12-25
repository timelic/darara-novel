<script lang="ts">
  import { onMount } from "svelte";
  import { text } from "./text";
  import { debounce } from "lodash-es";
  const texts = text.replaceAll("“", "「").replaceAll("”", "」").split("\n");
  const fontSize = 18;
  const marginTop = 2 * fontSize;
  const marginBottom = 2 * fontSize;

  let lineHeight = 2 * fontSize;
  let elLeftTranslateY = 0;
  let elRightTranslateY = 0;

  let leftPageIndex = 0;
  let rightPageIndex = 1;

  let maxPageIndex = 100;
  onMount(() => {
    const elLeft = document.querySelector("#left")!;
    let screenHeight = window.innerHeight - marginTop - marginBottom;
    const calcSize = () => {
      console.log("init");
      screenHeight = window.innerHeight - marginTop - marginBottom;
      maxPageIndex = Math.floor(elLeft.scrollHeight / screenHeight / 2) + 1;
      const lines = Math.floor(screenHeight / lineHeight);
      lineHeight = screenHeight / lines;
    };
    window.onresize = debounce(calcSize, 1000);
    calcSize();
    elRightTranslateY = -screenHeight * rightPageIndex;
    document.onwheel = (e) => {
      const direction = e.deltaY > 0 ? 1 : -1;
      // 这有 bug 不想改了
      if (
        leftPageIndex + direction < 0 ||
        rightPageIndex + direction > maxPageIndex
      ) {
        return;
      }
      leftPageIndex += direction;
      rightPageIndex += direction;
      elLeftTranslateY = -leftPageIndex * screenHeight * 2;
      elRightTranslateY = -rightPageIndex * screenHeight * 2;
    };
  });
</script>

<div id="header">
  <div style="display: flex; column-gap: 2rem;">
    <div class="page-number">{leftPageIndex + 1}</div>
    <div>异世界奶茶店</div>
  </div>
  <div class="page-number">{rightPageIndex + 1}</div>
</div>

<div
  id="chapter-container"
  style="--line-height: {lineHeight}px; font-size: {fontSize}"
>
  <div id="left">
    <div class="wrap" style="--translateY: {elLeftTranslateY}">
      {#each texts as text}
        <p data-start="{text.charAt(0)}">{text}</p>
      {/each}
    </div>
  </div>
  <div id="right">
    <div class="wrap" style="--translateY: {elRightTranslateY}">
      {#each texts as text}
        <p data-start="{text.charAt(0)}">{text}</p>
      {/each}
    </div>
  </div>
</div>

<style lang="scss">
  p[data-start="「"] {
    // 嘛 为了首字对齐
    text-indent: -0.5rem;
  }
  #chapter-container {
    display: flex;
    flex-direction: row;
    height: calc(100vh - var(--margin-top) - var(--margin-bottom));
    overflow: hidden;
    font-family: var(--font-seilf);
    font-size: var(--font-size);
    line-height: var(--line-height);
    padding: 0 var(--padding-x);
    column-gap: 6rem;
    #left,
    #right {
      flex: 1;
      .wrap {
        display: flex;
        flex-direction: column;
        // row-gap: calc(var(--line-height) / 1);
        transform: translateY(calc(var(--translateY) * 1px));
      }
    }
  }
  #header {
    position: fixed;
    top: 0;
    height: var(--margin-top);
    line-height: var(--margin-top);
    // 往下挪挪
    transform: translateY(calc(0.1 * var(--margin-top)));
    display: flex;
    justify-content: space-between;
    padding: 0 var(--padding-x);
    width: calc(100vw - 2 * var(--padding-x));
    font-family: var(--font-seilf);
    font-size: small;
  }
</style>
