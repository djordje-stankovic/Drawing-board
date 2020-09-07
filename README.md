## Drawing-board component for vue.js


# Usage 
```
 <DrawingBoard> </DrawingBoard>
```
## Props which receives
```
 <DrawingBoard 
   :colorSettings
   :canvasSeting
   :colors
>
 </DrawingBoard>
```
### Default Canvase Seting
```
canvasSetting : {
  width: 300,
  height: 300,
  border: {
       color: "black",
       style: "solid ",
       size: 2,
        },
    }
  }
    
```
### Default Color Seting
```
  colorSettings : {
   margin: 2,
   colorWidth: 15,
   colorHeight: 15,
   background: "White",
}
```
### Default List of Color
```
 colors : [
          "#FC2C00",
          "#5FFC00",
          "#00FCCE",
          "#0400FC",
          "#FC00F8",
          "#FC006B",
          "#FCBB00",
          "#00FC63",
          "#008DFC",
          ]
}
```



