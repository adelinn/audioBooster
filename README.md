# audioBooster
This is a javascript project that aims at increasing the sound volume by playing it on all speakers.

```javascript
function makeIFrame() {
    let ifrm = document.createElement("iframe");
    //ifrm.setAttribute("src", location.href); //TODO: change with own domain if possible
    ifrm.style.display = "none";
    document.body.appendChild(ifrm);
    //ifrm.contentWindow.document.close();
    //ifrm.contentWindow.document.write("");
    return ifrm.contentWindow;
}

function makeAudio(doc, streamObject, outputDeviceId) {
    let audio = doc.createElement("audio");
    audio.srcObject = streamObject;
    audio.style.display = "none";
    audio.setSinkId(outputDeviceId);
    doc.body.appendChild(audio);
    return audio;
}

function now() {
    return window.performance && window.performance.now ? ( performance.timeOrigin ? performance.timeOrigin : window.performance.timing.navigationStart ) + window.performance.now() : Date.now();
}

function getAudioSrc(){
    let src = document.querySelector("video") || document.querySelector("video"), s = src.captureStream();
    (s.getVideoTracks()).forEach(x=>{
        x.stop();
        s.removeTrack(x);
    });
    return {"src": src, "s": s};
}

// Main code
navigator.mediaDevices.getUserMedia({audio: true}).then(s=>{
    s.getTracks().forEach(x=>x.stop()); //stop mic use because we need only outputs
    navigator.mediaDevices.enumerateDevices().then(o=>{
        o = o.filter(({ kind, deviceId }) => kind === 'audiooutput' && deviceId !== 'communications');
        let defaultOutput = o.filter(({deviceId})=>deviceId==='default')[0];
        const outputs = o.filter(({ groupId }) => groupId !== defaultOutput.groupId);

        let audioSrc = getAudioSrc(), players = [];

        audioSrc.src.pause(); // Pause source to start audio in sync ?
        audioSrc.src.addEventListener("pause", () => players.forEach(x=>x.pause()));
        audioSrc.src.addEventListener("play", () => players.forEach(x=>x.play()));
        setInterval(()=>{audioSrc.src.pause();audioSrc.src.play();},5000); // TODO: sync with AudioDelay

        outputs.forEach((x)=>{
            let ifrm = makeIFrame(), audioEl = makeAudio(ifrm.document, audioSrc.s, x.deviceId);
            players.push(audioEl);
        });
    });
}).catch(e=>console.log(e));
```

This code was tested only on YouTube but any similar websites may work too.
