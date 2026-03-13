import React, { useState, useEffect } from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import * as Location from "expo-location";

export default function MeterScreen() {

  const baseFare = 70;
  const perKm = 25;
  const perMin = 5;

  const [running, setRunning] = useState(false);
  const [fare, setFare] = useState(baseFare);
  const [distance, setDistance] = useState(0);
  const [seconds, setSeconds] = useState(0);

  const [lastLocation, setLastLocation] = useState<any>(null);

  useEffect(() => {

    let timer:any;

    if (running) {

      timer = setInterval(() => {
        setSeconds((s)=>s+1)
      },1000)

      getLocation()

    }

    return ()=> clearInterval(timer)

  },[running])


  async function getLocation(){

    let { status } = await Location.requestForegroundPermissionsAsync()

    if(status !== "granted") return

    Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        distanceInterval: 10
      },
      (location)=>{

        if(lastLocation){

          const dx = location.coords.latitude - lastLocation.coords.latitude
          const dy = location.coords.longitude - lastLocation.coords.longitude

          const d = Math.sqrt(dx*dx + dy*dy) * 111

          setDistance(prev => prev + d)

        }

        setLastLocation(location.coords)

      }
    )

  }


  useEffect(()=>{

    const kmFare = distance * perKm
    const timeFare = (seconds/60) * perMin

    setFare(Math.floor(baseFare + kmFare + timeFare))

  },[distance,seconds])


  return(

    <View style={styles.container}>

      <Text style={styles.title}>Driver Meter</Text>

      <Text style={styles.data}>距離 {distance.toFixed(2)} km</Text>

      <Text style={styles.data}>時間 {Math.floor(seconds/60)} 分</Text>

      <Text style={styles.fare}>$ {fare}</Text>

      <View style={styles.buttons}>

        <Button
          title={running ? "暫停" : "開始"}
          onPress={()=>setRunning(!running)}
        />

        <Button
          title="結束"
          onPress={()=>{
            setRunning(false)
          }}
        />

      </View>

    </View>

  )

}

const styles = StyleSheet.create({

container:{
flex:1,
justifyContent:"center",
alignItems:"center"
},

title:{
fontSize:32,
fontWeight:"bold",
marginBottom:20
},

data:{
fontSize:22,
marginBottom:10
},

fare:{
fontSize:40,
color:"green",
marginTop:20
},

buttons:{
flexDirection:"row",
gap:20,
marginTop:30
}

})
