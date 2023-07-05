# propVR Globe Component

#Installation
1. npm i propvr-globe-component@latest --save

2. import {globeComp} from 'propvr-globe-component';

3. <globeComp :Data="gData" :markerIcon="markerSvg"  @emitClickData="emitClickData"/>
    
    Data -> Pass the list of city co-ordinates with corresponding ids.
    markerIcon -> Pass the icon's svg as string which will be used to indicate the city in the globe.
    
    emitclickData -> Will emit the city Id which is clicked in the globe by user. 

