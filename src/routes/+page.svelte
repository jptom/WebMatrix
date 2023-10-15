<script>
  import { browser } from '$app/environment';
  let screen;
  let isFullScreen = false;
  let n_words = 0;
  let n_lines = 0;
  let space_symbol = "&nbsp;";
  let line_texts = Array();
  let remaining_lengths = Array();
  let length = Array();
  let heads = Array();
  let html_lines = "";
  const oPTag = '<p class="vertical">';
  const cPTag = '</p>';
  const oHeadTag = '<span class="head">';
  const cHeadTag = '</span>'
  let height, width;
  let n;
  let index;
  let copy_element;
  let original_index;
  let x, y;
  let is_run=false;
  let id = 0;
  let change = false;
  let speed=40;
  let hidden = false;
  let asynchronous = false;
  let gradation = false;
  let pause = false;
  let colorNum = 0;
  let color = [0, 0, 0];
  const colorRGBs = [[0, 255, 0], 
                     [255, 0, 0],
                     [0, 0, 255],
                     [255, 255, 0],
                     [0, 255, 255],
                     [255, 0, 255],
                     [255, 255, 255]];
  const alphabet = "abcdefghijklmnopqrstuvwxyz"
  const numbers = "1234567890"
  const Alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  const symbols = "!@#$%^&*()`+|[]{};:\'\""
  const candidate_words = alphabet + Alphabet + numbers + symbols; 

  function responseResize() {
    console.log(window.innerHeight, window.innerWidth);
    clear_screen();
  }
  if (browser) {
    window.onresize = responseResize;
  }
  function clear_screen(){
    //console.log("clear_screen")
    if (browser) {
      let root = document.documentElement;
      console.log(window.innerWidth, window.innerHeight);
      width = window.innerWidth;
      height = window.innerHeight;
      let fontSize = window.getComputedStyle(root).getPropertyValue('font-size').slice(0, -2);
      console.log(fontSize)
      n_words = Math.floor(height / fontSize);
      n_lines = Math.floor(width / fontSize);
      console.log(n_words, n_lines);
    }
    line_texts = Array(n_lines).fill(null).map(()=>Array(n_words).fill(' '));
    length = Array(n_lines).fill(0);
    heads = Array(n_lines).fill(null).map(()=>Array(n_words).fill(0));
    remaining_lengths =  Array(n_lines).fill(null).map(()=>Math.floor(Math.random()*(n_words-1))+2);
    //console.log(remaining_lengths)
    const MIN = Math.min(...remaining_lengths);
    //console.log(MIN)
    remaining_lengths = remaining_lengths.map(element => element - MIN + 1);
    //console.log(remaining_lengths)
  }

  function html_rendering(){
    html_lines = line_texts.map((line, x) =>
      oPTag + line.map((char, y)=>{
        if(heads[x][y] == 0){
          return space_symbol;
        }
        else if(heads[x][y]==1){
          return !gradation? oHeadTag+char+cHeadTag : "</span>"+oHeadTag+char+cHeadTag;
        }
        else{
          if(!gradation){
            return char;
          }
          return (y==0 | heads[x][y-1]==0)? `<span style=background:linear-gradient(0,rgb(${color[0]},${color[1]},${color[2]}),rgb(${Math.floor(color[0]*heads[x][y])+50},${Math.floor(color[1]*heads[x][y])+50},${Math.floor(color[2]*heads[x][y])+50}));-webkit-background-clip:text;-webkit-text-fill-color:transparent;>`+char:char;
        }
      }).join('') + cPTag
    ).join('');
  }

  function move(){
    for(x=0; x<n_lines; x++){
      if(asynchronous & Math.random()<0.5){
        continue;
      }
      for(y=n_words-1; y>0; y--){
        if(line_texts[x][y-1]==' '){
          line_texts[x][y]=' ';
        }          
        else if(line_texts[x][y]==' '){
          line_texts[x][y] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
        }
        else if(change & Math.random()<0.05){
          line_texts[x][y] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
        }
        heads[x][y] = heads[x][y-1];
      }
      remaining_lengths[x]--;
      if(remaining_lengths[x]==0){
        remaining_lengths[x] = line_texts[x][0]==' '? Math.floor(Math.random()*(n_words-3))+4 : Math.floor(Math.random()*(n_words-1))+2;
        if(line_texts[x][0]==' '){
          line_texts[x][0] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
          heads[x][0] = 1;
          length[x] = remaining_lengths[x];
        }
        else{
          line_texts[x][0] = ' ';
          heads[x][0] = 0;
          length[x] = 0;
        }
      }
      else{
        heads[x][0] = length[x]? (remaining_lengths[x]+0.9)/length[x] : 0;
      }
    }
    

  }
  function update(){
    //console.log("update")
    //console.log(heads)
    if(!pause){
      move();
      html_rendering();
    }
    setTimeout(update, speed);
    //console.log(line_texts)
  }
  function onKeyDown(event){
    switch(event.key){
      case '1':
      case '2':
      case '3':
      case '4':
      case '5':
      case '6':
      case '7':
      case '8':
      case '9':
        speed = Number(event.key)*10;
        if(asynchronous){
          speed = speed/1.41
        }
        break;
      case 'a':
        asynchronous = !asynchronous;
        speed = asynchronous? speed/1.41 : speed*1.41;
        break;
      case 'c':
        colorNum = (colorNum+1)%colorRGBs.length;
        color = colorRGBs[colorNum];
        break;
      case 'd':
        change = false;
        asynchronous = false;
        gradation = false;
        speed = 40;
        break;
      case 'f':
        break;
        if(!isFullScreen){
          screen.requestFullscreen();
          isFullScreen = true;
        }
        else{
          document.exitFullscreen();
          isFullScreen = false;
        }
        break;
      case 'g':
        gradation = !gradation
        break;
      case 'k':
        change = !change;
        break;
      case 'l':
        break;
      case 'h':
        hidden = !hidden;
        break;
      case 'p':
        pause = !pause;
        break;
      case 'r':
        clear_screen();
        break;
      }
      html_rendering();
    }
  /*-- main --*/
  color = colorRGBs[colorNum];
  clear_screen();
  html_rendering();
  update();
</script>

<svelte:head>
  <title>Matrix</title>
	<meta name="description" content="Matrix written in SvelteKit" />
</svelte:head>

<dev class="screen" style="color: rgb({color[0]},{color[1]},{color[2]});" bind:this={screen}>
  {@html html_lines}
</dev>
<dev class="description" class:hidden>
  <h1>Web-Matrix</h1>
  <p class="description_content">This web page simulate the display from "The Matrix".<br>
    User can control scrolling from keyboard.<br> 
    This program is written using Svelte based on 
    <a href="https://github.com/abishekvashok/cmatrix" target="_blank">CMatrix</a> and 
    <a href="https://github.com/will8211/unimatrix" target="_blank">UniMatrix</a>.
  </p>
  <h2 class="description_content">KEYBOARD CONTROL:</h2>
  <p class="description_content key_control">
    1~9 : scrolling speed.<br>
    a : Asynchronous scroll. Lines will move at varied speeds.<br>
    c : Change color.<br>
    d : Apply default setting.<br>
    f : Full screen mode. (🚧)<br>
    g : Gradation scrolling.<br>
    h : Show or hide descripton.<br>
    k : Characters change while scrolling.<br>
    l : Change character set. (🚧)<br>
    p : Pause.<br>
    r : Restart scrolling.</p>
</dev>
<svelte:window on:keydown|preventDefault={onKeyDown} />
<style>
  
  .screen {
    writing-mode: vertical-rl;
    text-orientation: upright;
    background-color: rgb(0, 0, 0);
  }
  .screen > :global(.vertical) {
    margin: 0%;
    width: 18px;
  }
  .screen > :global(.vertical) > :global(.head) {
    color: rgb(255, 255, 255);
  }
  .description{
    visibility: visible;
    position: absolute;
    border: 5px solid #ffffff;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: 10% auto auto auto;
    border-radius: 10px; 
    width:500px;
    height:500px;
    background-color: rgb(250, 150, 50);
  }
  .description.hidden{
    visibility: hidden;
  }
  h1 {
    text-align: center;
  }
  .description_content{
    padding-right: 20px;
    padding-left: 20px;
  }
  .key_control{
    line-height: 1.5em;
  }
</style>

