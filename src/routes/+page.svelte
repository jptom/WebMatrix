<script>
  import { browser } from '$app/environment';
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
  let speed=0;
  let hidden = false;
  let asynchronous = false;
  let gradation = false;
  const alphabet = "abcdefghijklmnopqrstuvwxyz"
  const numbers = "1234567890"
  const Alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  const symbols = "!@#$%^&*+"
  const candidate_words = alphabet + Alphabet + numbers + symbols; 

  if (browser) {
    let root = document.documentElement;
    
    console.log(window.innerWidth, window.innerHeight);
    width = window.innerWidth;
    height = window.innerHeight;
    let fontSize = window.getComputedStyle(root).getPropertyValue('font-size').slice(0, -2);
    console.log(fontSize)
    n_words = Math.floor(height / fontSize);
    n_lines = Math.floor(width / fontSize);
    console.log(n_words, n_lines)
  }

  function clear_screen (){
    console.log("clear_screen")
    line_texts = Array(n_lines).fill(null).map(()=>Array(n_words).fill(' '));
    length = Array(n_lines).fill(0);
    heads = Array(n_lines).fill(null).map(()=>Array(n_words).fill(0));
    remaining_lengths =  Array(n_lines).fill(null).map(()=>Math.floor(Math.random()*(n_words-1))+2);
    console.log(remaining_lengths)
    const MIN = Math.min(...remaining_lengths);
    console.log(MIN)
    remaining_lengths = remaining_lengths.map(element => element - MIN + 1);
    console.log(remaining_lengths)
  }

  function html_rendering(){
    html_lines = line_texts.map((line, x) =>
      oPTag + line.map((char, y)=>{
        return char==' '? space_symbol : heads[x][y]==1? oHeadTag+line[y]+cHeadTag : gradation? `<span style=opacity:${heads[x][y]}>`+line[y]+'</span>':line[y]
      }).join('') + cPTag
    ).join('');
  }

  function move(){
    for(x=0; x<n_lines; x++){
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
    }
    for(x=0;x<n_lines; x++){
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
    move()
    html_rendering()
    //console.log(line_texts)
  }
  function changeSpeed(speed){
    clearInterval(id);
    id = setInterval(update, speed);
  }
  function onKeyDown(event){
    switch(event.key){
      case '1':
        speed = 10;
        changeSpeed(10);
        break;
      case '2':
        speed = 20;
        changeSpeed(20);
        break;
      case '3':
        speed = 30;
        changeSpeed(30);
        break;
      case '4':
        speed = 40;
        changeSpeed(40);
        break;
      case '5':
        speed = 50;
        changeSpeed(50);
        break;
      case '6':
        speed = 60;
        changeSpeed(60);
        break;
      case '7':
        speed = 70;
        changeSpeed(70);
        break;
      case '8':
        speed = 80;
        changeSpeed(80);
        break;
      case '9':
        speed = 90;
        changeSpeed(90);
        break;
      case 'a':
        asynchronous = !asynchronous;
      case 'd':
        change = false;
        asynchronous = false;
        speed = 40;
      case 'g':
        gradation = !gradation
        break;
      case 'k':
        change = !change;
        break;
      case 'h':
        hidden = !hidden;
        break;
      case 'p':
        if(is_run){
          clearInterval(id);
          is_run = false;
          console.log(line_texts)
          console.log(heads)
        } 
        else{
          id = setInterval(update, speed);
          is_run = true;
        }
        break;
      case 'r':
        clearInterval(id);
        clear_screen();
        html_rendering();
        id = setInterval(update, speed);
        is_run = true;
        break;
      }
  }
  /*-- main --*/
  
  clear_screen();
  html_rendering();
  speed = 40;
  id = setInterval(update, speed);
  is_run = true;

</script>

<svelte:head>
  <title>Matrix</title>
	<meta name="description" content="Matrix written in SvelteKit" />
</svelte:head>

<dev class="screen vertical">
  <!--<span class="vertical">matrix</span>-->
  {@html html_lines}
</dev>
<dev class="description" class:hidden>
  web-matrix
  KEYBOARD CONTROL:
  d : Apply default setting.
  k : Characters change while scrolling.
  r : Restart scrolling.

</dev>
<svelte:window on:keydown|preventDefault={onKeyDown} />
<style>
  
  .screen {
    writing-mode: vertical-rl;
    text-orientation: upright;
    background-color: rgb(0, 0, 0);
  }
  .screen > :global(.vertical) {
    color: rgb(0, 255, 0);
    margin: 0%;
  }
  .screen > :global(.vertical) > :global(.head) {
    color: rgb(255, 255, 255);
  }
  .description{
    visibility: visible;
    position:fixed;
    top:30px;
    left:30px;
    width:100px;
    height:100px;
    background-color: rgb(255, 149, 50);
  }
  .description.hidden{
    visibility: hidden;
  }
</style>

