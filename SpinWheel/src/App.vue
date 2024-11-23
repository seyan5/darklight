
<template>
  <div class="app-container">
    <div class="header">
      <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@800&display=swap" rel="stylesheet">
      <nav>
        <div class="left-items">
          <img src="/src/assets/logo.png" alt="Logo" style="width: 38px; height: 38px; border-radius: 50%; margin-right: 8px;">
        <span>wheelofnames.com</span> <!-- Just a label, no link -->
    </div>
    <div class="right-items">
    <a>
      <i class="bi bi-palette-fill" style="color: white; font-size: 16px;"></i>
        Customize
    </a>
    <a>
      <i class="bi bi-file-earmark-fill" style="color: white; font-size: 16px;"></i>
       New
    </a>
    <a> <i class="bi bi-folder-fill" style="color: white; font-size: 16px;"></i>
      Open
    </a>
    <a> <i class="bi bi-floppy-fill" style="color: white; font-size: 16px;"></i>
       Save
    </a>
    <a><i class="bi bi-share-fill" style="color: white; font-size: 16px;"></i>
       Share
    </a>
    <a><i class="bi bi-search" style="color: white; font-size: 16px;"></i>
       Gallery
    </a>
    <a><i class="bi bi-fullscreen" style="color: white; font-size: 16px;"></i>
    </a>
    <a> 
       More
       <i class="bi bi-caret-down-fill" style="color: white; font-size: 14px;"></i>
    </a>
    <a> <i class="bi bi-globe2" style="color: white; font-size: 16px;"></i>
       English
    </a>
  </div>
</nav>
    </div>

    <div class="hide-wrapper">
    <input type="checkbox" class="hide" v-model="isHidden"> Hide
  </div>
  

    <div class="tabbed" :class="{ hidden: isHidden }">
  <input type="radio" name="tabs" id="tab-nav-1" checked>
  <label for="tab-nav-1">Entries</label>
  <input type="radio" name="tabs" id="tab-nav-2">
  <label for="tab-nav-2">Results</label>

  <div class="tab-content"  id="tab1">
   <div class ="buttons">
    <button type="button" class="shuffle" name="myButton">
      <i class="bi bi-shuffle" style="color: white; font-size: 16px;"></i>
   Shuffle
</button>
  <button type="button" class="sort" name="myButton">
  <i class="bi bi-sort-alpha-up"style="color: white; font-size: 18px;"></i>
   Sort
  </button>
  <button type="button" class="image" name="myButton">
  <i class="bi bi-image-fill" style="color: white; font-size: 16px;"></i>
   Add image
   <i class="bi bi-caret-down-fill" style="color: white; font-size: 10px;"></i>
  </button>
  <div class="advanced-wrapper" >
  <input type="checkbox" name="advanced" id="advanced" :class="{ hidden: isHidden }">
  <label for="advanced" :class="{ hidden: isHidden }">Advanced</label>
</div>
   </div>

   <textarea class="entry-input1" v-model="wheelItems" @input="updateWheel" placeholder="this is tab 1"></textarea>
    <div class="footer">
    <text class="version">Version 312</text>
    <text class="changelog">Changelog</text>
  </div>
  </div>

  <div class="tab-content" id="tab2">
    <div class ="buttons">
<button type="button" class="sort" name="myButton">
  <i class="bi bi-sort-alpha-up"style="color: white; font-size: 18px;"></i>
   Sort
</button>
<button type="button" class="clear" name="myButton" id="clear" @click="clearResultList">
  <i class="bi bi-x" style="color: white; font-size: 20px;"></i>
  Clear the list
</button>
</div>
 <textarea class="entry-input2" placeholder="" readonly></textarea>
   <div class="footer">
    <text class="version">Version 312</text>
    <text class="changelog">Changelog</text>
  </div>
  </div>

</div>


    <div class="wheel-container">
      <div class="svg-container" id="fadeArea">
            <svg viewBox="5 0 500 500" id="function-fade">
                <path id="text" d="M 130, 220 A 250,250 0 0,1 450,250" fill="transparent"></path>
                <g transform="translate(0, -110)">
                <text font-size="40px" font-family="sans-serif" font-weight="800" letter-spacing="5px">
                    <textPath href="#text" startOffset="10px" >
                        Click to spin
                    </textPath>
                </text>
              </g>

                <path id="text2" d="M 100, 250 A 200,200 0 0,0 450,250" fill="transparent"></path>
                <g transform="translate(0, 130)">
                <text font-size="40px" font-family="sans-serif" font-weight="800" letter-spacing="5px">
                    <textPath href="#text2" startOffset="10px" >
                        or press ctrl+enter
                    </textPath>
                </text>
              </g>
            </svg>
        </div>
        <div class="triangle"></div>
      <div class="wheel">
        <canvas ref="canvas" width="500" height="500"></canvas>
      <div class="center-circle">  
       </div>
     </div>
    </div>
  </div>
</template>


<script>
import spinSoundFile from "@/assets/spin.mp3";
export default {
  data() {
    return {
      wheelItems: "DARK\nLIGHT\nDARK\nLIGHT\nDARK\nLIGHT", // default items
      name: "MyComponent",
      winner: " ", // Initial winner
      currentDeg: 0,
      step: 0,
      itemDegs: {},
      speed: 0,
      maxRotation: 0,
      pause: false,
      items: [],
      result: [],
      spinSound: null,
      isIdle: true, // Controls whether the idle animation is active
      isSpinning: false,
      isHidden: false // Controls the visibility of the tabs
    };
  },
  computed: {
    canvas() {
      return this.$refs.canvas;
    },
  },
  methods: {
    updateWheel() {
      this.createWheel();  // Recreate the wheel with new items whenever the input changes
    },
    toRad(deg) {
      return deg * (Math.PI / 180.0);
    },
    randomRange(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    easeOutSine(x) {
      return Math.sin((x * Math.PI) / 2);
    },
    getPercent(input, min, max) {
      return (((input - min) * 100) / (max - min)) / 100;
    },
    createWheel() {
      this.items = this.wheelItems.split("\n");
      this.step = 360 / this.items.length;
      this.draw();
    },
    draw() {
      const ctx = this.canvas.getContext("2d");
      const width = this.canvas.width;
      const height = this.canvas.height;
      const centerX = width / 2;
      const centerY = height / 2;
      const radius = width / 2;

      ctx.clearRect(0, 0, width, height); // Clear the canvas

      ctx.beginPath();
      ctx.arc(centerX, centerY, radius, this.toRad(0), this.toRad(360));
      ctx.fillStyle = `rgb(${33},${33},${33})`;
      ctx.lineTo(centerX, centerY);
      ctx.fill();

      let startDeg = this.currentDeg;
      for (let i = 0; i < this.items.length; i++, startDeg += this.step) {
        let endDeg = startDeg + this.step;

        // Alternate Black and White colors
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius - 2, this.toRad(startDeg), this.toRad(endDeg));
        ctx.fillStyle = i % 2 === 0 ? "black" : "white";
        ctx.lineTo(centerX, centerY);
        ctx.fill();

        // Draw text
        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(this.toRad((startDeg + endDeg) / 2));
        ctx.textAlign = "center";
        ctx.fillStyle = i % 2 === 0 ? "white" : "black";
        ctx.font = ' 65px Calibri Light';
        ctx.fillText(this.items[i], 160, 30, 200);
        ctx.restore();

        this.itemDegs[this.items[i]] = { startDeg, endDeg };

        // Check winner
        if (startDeg % 360 < 360 && startDeg % 360 > 270 && endDeg % 360 > 0 && endDeg % 360 < 90) {
          this.winner = this.items[i];
        }
      }
    },
    animate() {
      if (this.pause) return;
      this.speed = this.easeOutSine(this.getPercent(this.currentDeg, this.maxRotation, 0)) * 15;
      if (this.speed < 0.01) {
        this.speed = 0;
        this.pause = true;
        this.result.unshift(this.winner);
        console.log('Current Result:', this.result);
        console.log('Current Winner:', this.winner);

        // Get the textarea element and update its value with the results
        const entryInput2 = document.querySelector('.entry-input2');
        if (entryInput2) {
          // Join the results as a string, each result on a new line
          entryInput2.value = this.result.join('\n');
        } else {
          console.log('Textarea element not found!');
        }
      }
      this.currentDeg += this.speed;
      this.draw();
      window.requestAnimationFrame(this.animate);
    },
    clearResultList() {
      let e1 = document.querySelector('.entry-input2');
      e1.value = "";
      // Clear the result array and reset resultText
      this.result = [];
      this.resultText = "";
    },
    spin() {
      if (this.speed !== 0) return;
      if (this.spinSound) {
        this.spinSound.currentTime = 0; // Reset to the start of the audio
        this.spinSound.play(); // Play the spin sound
      }

      this.createWheel();
      this.draw();

      // Pick a random item as the target
      const randomItem = this.items[Math.floor(Math.random() * this.items.length)];
      const targetDeg = this.itemDegs[randomItem].endDeg;

      // Set maxRotation for several rotations before landing on the target
      this.maxRotation = 360 * 2 + targetDeg;
      this.currentDeg = 0;
      this.pause = false;

      // Add the winner to history when the spin stops
      // add to the start of the list

      // Start animation
      window.requestAnimationFrame(this.animate);
    },
    winBLACK() {
      if (this.speed !== 0) return;

      const sectionAngle = 360 / 6;  // assuming 6 sections for alternating BLACK and WHITE
      const blackSections = [1, 3, 5];  // BLACK sections

      // Choose a random BLACK section
      let randomSection = blackSections[Math.floor(Math.random() * blackSections.length)];

      // Use your preferred landing spots as possible offsets
      let possibleOffsets = [
        365.5 * 5,
        358.5 * 5,
        358 * 5,
        357.8 * 5,
        360.8 * 5,
        361.1 * 5,
        361.5 * 5,
        361.9 * 5,
        362.3 * 5,
        362.9 * 5,
        354.9 * 5,
        354.7 * 5,
        355.8 * 5
      ];

      // Randomly select one of the preferred offsets
      let randomOffset = possibleOffsets[Math.floor(Math.random() * possibleOffsets.length)];

      // Calculate the final angle for the BLACK section, adding the chosen offset
      const finalAngle = randomSection * sectionAngle + (sectionAngle / 2) + randomOffset;

      // Set up for spinning with multiple rotations
      this.maxRotation = finalAngle + 360 * 1;
      this.currentDeg = 0;
      this.pause = false;
      if (this.spinSound) {
        this.spinSound.currentTime = 0; // Reset to the beginning
        this.spinSound.play();
      }

      // Add the winner to history when the spin stops

      // Start animation
      window.requestAnimationFrame(this.animate);
    },

    winWHITE() {
      if (this.speed !== 0) return;

      const sectionAngle = 360 / 6;  // assuming 6 sections for alternating BLACK and WHITE
      const whiteSections = [0, 2, 4];  // WHITE sections

      // Choose a random WHITE section
      let randomSection = whiteSections[Math.floor(Math.random() * whiteSections.length)];

      // Use the same preferred landing spots for WHITE as well
      let possibleOffsets = [
        365.5 * 5,
        358.5 * 5,
        358 * 5,
        357.8 * 5,
        360.8 * 5,
        361.1 * 5,
        361.5 * 5,
        361.9 * 5,
        362.3 * 5,
        362.9 * 5,
        354.9 * 5,
        354.7 * 5,
        355.8 * 5
      ];

      // Randomly select one of the preferred offsets
      let randomOffset = possibleOffsets[Math.floor(Math.random() * possibleOffsets.length)];

      // Calculate the final angle for the WHITE section, adding the chosen offset
      const finalAngle = randomSection * sectionAngle + (sectionAngle / 2) + randomOffset;

      // Set up for spinning with multiple rotations
      this.maxRotation = finalAngle + 360 * 1;
      this.currentDeg = 0;
      this.pause = false;
      if (this.spinSound) {
        this.spinSound.currentTime = 0; // Reset to the beginning
        this.spinSound.play();
      }

      // Add the winner to history when the spin stops
      // Start animation
      window.requestAnimationFrame(this.animate);
    },
    handleKeyPress(event) {
      if (event.key === 'z' || event.key === 'Z') {
        this.winBLACK();
        if (fadeArea) {
          fadeArea.classList.add('hidden');
        }

      }
      if (event.key === 'x' || event.key === 'X') {
        this.winWHITE();

        if (fadeArea) {
          fadeArea.classList.add('hidden');
        }
      }
    }
  },
  mounted() {
    this.createWheel();
    window.addEventListener('keypress', this.handleKeyPress);

    //SOUND
    this.spinSound = new Audio(spinSoundFile);
    this.spinSound.preload = "auto";
  },
};
</script>


<style>
* {
    border: 0;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
#advanced {
  cursor: pointer;
  position: relative; /* Allows positioning relative to its normal position */
  top: -60px; /* Moves the checkbox down by 10px, adjust this value as needed */
  left: 10px;
}
#fadeArea {
    opacity: 1;
    transition: opacity .1s ease-out; /* Adjust duration as needed */
}

#fadeArea.hidden {
    opacity: 0;
    pointer-events: none; /* Prevent interactions while fading out */
}

@media (max-width: 1080px) and (max-height: 1920px) {
  .wheel {
    width: 100%; /* Make the wheel scale with the viewport */
    height: auto;
    }
}
  
  

.hidden {
    display: none;
  }
.svg-container {
  filter: drop-shadow(10px 10px 12px #000000);
                              
  position: absolute; /* Moves the element out of the normal document flow */
  left: 345px; 
  top:150px;/* Optional: aligns it to the left of the containing element */
  width: 70%; /* Optional: sets the width to full width if needed */
  z-index: 11; /* Ensures it stays on top of lower z-index elements */
  fill: white; /* Makes all text within the SVG white */
}
.svg-container text {
  font-weight: 800 !important;
}

.advanced-wrapper {
  z-index: 11;
  position: absolute;
  top: 100px; /* Adjust this value to move it up */
  right: 150px;
  cursor: pointer;
  color: white;
  font-size: 10px !important;
  display: flex; /* Aligns checkbox and label horizontally */
  align-items: center; /* Vertically align the checkbox and label */
   /* Adds space between the checkbox and label */
}

.hide-wrapper {
  position:absolute;
  top: 70px; /* Adjust this value to move it up */
  right:35px;
  cursor: pointer;
  color: white;
}
.buttons {
  display: flex; /* Use flexbox to align buttons next to each other */
  gap: 10px; /* Add some space between the buttons */
  margin-top: -60px;
}
.shuffle, .sort, .image, .clear{
  display:inline-flex; /* Use flexbox to align items horizontally */
  align-items: center; /* Vertically centers the items (icon and text) */
  padding: 2px 10px;
  background: rgba(255, 255, 255, 0.1); /* Slightly transparent white background */
  border-radius: 5px; /* Rounded corners */
  backdrop-filter: blur(10px); /* Apply blur effect to the background */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Add subtle shadow for more depth */
  color: white; /* White text color for contrast */
  font-size: 13px;
  gap:10px;
  font-weight: bold;
  margin-top: 10px;
}
.shuffle:hover{
  color: #ffffff;
  background: rgba(245, 244, 244, 0.205);
}
.sort:hover{
  color: #ffffff;
  background: rgba(245, 244, 244, 0.205);
}
.image:hover{
  color: #ffffff;
  background: rgba(245, 244, 244, 0.205);
}
.clear:hover{
  color: #ffffff;
  background: rgba(245, 244, 244, 0.205);
}

.shuffle, .sort, .image i {
  margin-right: -5px; /* Add space between icon and label */
}

.footer {
  display: flex;
  margin-top: -20px;
  justify-content: space-between; /* Align the items on opposite sides */
  width: 100%; /* Ensure the footer takes up full width */
  font-size: 17px;
  color: #ffffff;
}
.changelog {
  text-align: right;
  color: #6ba9ec; /* Set to blue */
  text-decoration: underline;
}
.tabbed {
  width: 370px;
  margin: 80px auto;
  position: absolute;
  right: 20px;
  padding: 15px;
  background: #1d1b1b;
  border: 1px solid #666464;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  color: white;
}

.tabbed input[type="radio"] {
  display: none;
}

.tabbed label {
  display: inline-block; /* Change to inline-block to align side-by-side */
  margin-bottom: 0px;
  text-align: center;
  padding: 11px 50px; /* Add padding to space the labels out */
  cursor: pointer;
  position: relative;
  top: -60px;
  right: 30px;
  font-size: 16px;
  font-weight: 500;
  transition: color 0.5s ease, background 0.5s ease;
}

.tabbed label:hover {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}

.tabbed input[type="radio"]:checked + label {
  color: rgb(253, 253, 253);
  border-bottom: 2px solid white;
  padding-bottom: 8px;
}

.tab-content {  
  display: none; /* Hide all tab content by default */
}

#tab-nav-1:checked ~ #tab1,
#tab-nav-2:checked ~ #tab2 {
  opacity: 0;
  transform: translateX(0);
  display: block; /* Show only the checked tab's content */
}

#tab-nav-1:checked ~ #tab1,
#tab-nav-2:checked ~ #tab2 {
  opacity: 1;
  transform: translateX(0); /* Slide out of view */
}

#tab-nav-1:checked ~ #tab2,
#tab-nav-2:checked ~ #tab1 {
  opacity: 1;
  transform: translateX(-100%); /* Slide out of view */
}
.entry-input1, .entry-input2 {
  width: 100%;
  height: 480px;
  padding: 10px;
  background-color: transparent;
  border: 1px solid #645b5b;
  margin-top: 50px;
  margin-bottom: 20px;
  border-radius: 5px;
  font-size: 16px;
  resize: none;
}

nav .left-items {
    color: #faf9f9 !important;
    gap: 1px;
    margin-left: 1%;
    display: flex;
    align-items: center;
    font-family: 'Roboto', -apple-system, 'Helvetica', sans-serif;
    cursor: pointer;
    font-size: 21px;
    font-weight: 500;
}
nav .right-items {
    color: #faf9f9 !important;
    align-items: center;
    gap: -50px;
    display: flex;
    height: 100%;
    justify-content: flex-end; /* Right side items aligned to the right */
    align-items: center;
    font-family: 'Roboto', -apple-system, 'Helvetica', sans-serif;
    font-size: 16px;
    font-weight: 500;
}
nav {
    background: #3369e8;
    background-color: #3369e8;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    height: 50px;
    margin: 0;
    box-sizing: border-box;
    filter: drop-shadow(0px 0px 4px rgba(209, 207, 207, 0.5));
}

nav a {
    color: #faf9f9 !important;  /* Ensures white font */
    text-decoration: none;
    padding: 18px;
    display: flex;
    align-items: center;
    gap: 8px;
    height: 100%;
    cursor: pointer;
    font-family: 'Roboto', -apple-system, 'Helvetica', sans-serif;
    font-size: 16px;
    font-weight: 500;
}

nav a:hover {
    color: #ffffff;
    background: rgba(255, 255, 255, 0.1); /* Semi-transparent white */
    backdrop-filter: blur(10px);
}

body {
  font-family: Arial, sans-serif;
  background-color: #1d1c1c !important;
  position: relative;
  overflow-x: hidden;
  overflow-y:hidden;
}
.app-container {
  text-align: center;
}
.winner{
  position:absolute;
  text-align: right;
  left: 46%;
  margin-top: -75px;
}
@keyframes idleSpin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(60deg);
  }
}
.wheel-container {
  position: relative;
  display: inline-block;
}
@keyframes spinContinuous {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes rotateWheel {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.wheel {
  animation: rotateWheel 1s linear infinite;
}

.wheel {
  z-index: 1;
  display: flex;
  justify-content: center;
  position: relative;
  left: 200px; /* Keep the horizontal centering */
  margin-top: 50px;
  margin-bottom: 25px;
  width: 50vw;
  height: 50vw;
  max-width: 700px; /* Limit maximum size */
  max-height: 700px;
  transform: translateX(-50%); /* Adjust the position by translating it by half its width to maintain centering */
  border-radius: 50%; /* Ensure it remains circular */
  animation: idleSpin 4s linear 1 forwards; 
  animation: spinContinuous 4s linear 1 forwards;
  animation-duration: 6100ms;
}
@media (min-width: 1400px) {
  .wheel {
    left: 240px;
    width: 700px;
    height: 650px;
  }
  
}
.center-circle {
  width: 100px;
  height: 100px;
  border-radius: 60px;
  background-color: #ffffff;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}

.triangle {
  width: 0;
  height: 0;
  border-top: 25px solid transparent;
  border-bottom: 25px solid transparent;
  border-right: 50px solid rgb(200, 200, 200); /* Pointer color */
  position: absolute;
  top: 50%; /* Center vertically */
  right: -240px; /* Position just outside the wheel's right side */
  transform: translateY(-50%); /* Center horizontally */
  z-index: 10; /* Keeps it on top of the wheel */
  /* Add glowing box shadow */
  /* White glow effect */
  
}


textarea {
  background-color: rgba(20, 20, 20, 0.2);
  caret-color: #fff;
  resize: none;
  color: #fff;
}

</style>



