import React, {Component} from 'react';
import {View, StyleSheet, Text, Image, AppState} from 'react-native';
import {getNasabah, setLoading} from '../redux/action/NasabahAction';
import {getPinjaman, getPinjaman2, getPinjaman3, delPinjaman, delPstatNull, getHisPinjaman} from '../redux/action/PinjamanAction';
// import {getVirtualAcc } from '../redux/action/VirtualAcc_Action';
import {getVirtualAccount} from '../redux/action/VirtualAction';
import {connect} from 'react-redux';

import deviceStorage from '../services/deviceStorage';
import MainFresh from './components/MainFresh';
import MainFresh2 from './components/MainFresh2';
import NewLogin from './LoginScreen';
import FrmPinjam from './components/FrmPinjam';
import FormVirtualAcc from './components/FormVirtualAcc';
import {Header, Btn} from './commons/';
import Spinner from 'react-native-loading-spinner-overlay';
import AsyncStorage from '@react-native-community/async-storage';
import axios from 'axios';
import {IMAGE} from '../constants/Image';
import Moment from 'moment';
import { format } from "date-fns";
import BackgroundTimer from 'react-native-background-timer';
import {getAbout} from '../redux/action/AboutAction';

class MainScreen extends Component {
  constructor(props) {
    super(props);

    this.state = {
      jwt: null,
      myStat: null,
      loading: true,
      loadNasabah: 'false',
      besarPinjam: '500000',
      tujuanPinjam: '',
      tanggalPinjam: '',    
      statsPjm: '',
      errCode: null,
      errCode3: null,
      hisErrCode: null,
      img: require('../assets/images/userNotFound.png'),
      imgWait: require('../assets/images/userWaiting.png'),
      usrWait: require('../assets/images/manwait2.png'),
      bnkname: 'BCA',
      statVirtual: 'null',
      bunga: 0, 
      biaya_admin: 0,
      tenorPinjam: 0,
      hitungan: 0,
      N_mobileNo: '',
      N_tglPinjam: '',
      N_jmlPinjam: '',
      N_tujuanPinjam: '',
      N_tglBayar: '',
      N_bunga: 0,
      N_biaya_Admin: 0,
      N_tenorPinjam: 0,
      N_tglJatuhTempo: '',
      N_jmlBayar: 0,
      N_status_pinjam: 0,
      N_createdAt: '',
      N_updatedAt: '',
      historyExist: 0,
      rePinjam: 0,
      btnBayar: false,
      pinjLagi_Aktif: false,
      appState: AppState.currentState,
      errCodeLogMain: null,
      myIDMain: 17,
      myLogoutMain: '',
    };

    this.loadJWT = this.loadJWT.bind(this);
    this.loadJWT();
 
    this.pjmJWT = this.pjmJWT.bind(this);
    this.pjmJWT();

    this.approveJWT = this.approveJWT.bind(this);
    this.approveJWT();

    this.loadAbout = this.loadAbout.bind(this);
    this.loadAbout();

    this.loadMyStatus = this.loadMyStatus.bind(this);
    this.loadMyStatus();

    // this.slidingComplete = this.slidingComplete.bind(this);
    this.handlerChangeValue = this.handlerChangeValue.bind(this);
  }

  handlerChangeValue(stateName, stateValue) {
    this.setState({
      [stateName]: stateValue,
    });
  }

  async loadMyStatus() {
    try{
      const myValue = await AsyncStorage.getItem('myStatus');
      if(myValue !== null) {
        this.setState({
          myStat: myValue,
        });
        this.setState({
          rePinjam: 2,
        });
        console.log('Status Gue : ' + myValue);
      }
    }catch(error) {
      console.log('AsyncStorage My Status Error: ' + error.message);
    }
  }

  async loadJWT() {
    try {
      const value = await AsyncStorage.getItem('id_token');
      if (value !== null) {
        this.props.getNasabah(value);
        this.setState({
          jwt: value,
          loading: false,
        });
        this.cekStatus();
      } else {
        this.setState({
          loading: false,
        });
      }
    } catch (error) {
      console.log('AsyncStorage Error: ' + error.message);
    }
  }

  async pjmJWT() {
    try {
      const value = await AsyncStorage.getItem('id_token');
      if (value !== null) {
        // console.log('ini valuenya' + value);
        //  alert('persiapan pjmJWT');
        this.props.getPinjaman(value);
        this.setState({
          jwt: value,
          loading: false,
        });        
        // alert('pjmJWT nich' + this.state.jwt);
      } else {
        this.setState({
          loading: false,
        });
        // alert('loadingnya' + loading);
      }
    } catch (error) {
      console.log('AsyncStorage Status pinjam 1 Error: ' + error.message);
    }
  }

  async approveJWT() {
    try {
      const value = await AsyncStorage.getItem('id_token');
      if (value !== null) {
        // console.log('ini valuenya' + value);
        // alert('persiapan approveJWT');
        this.props.getPinjaman2(value);
        this.setState({
          jwt: value,
          loading: false,
        });
        
        // alert('jwt nich' + this.state.jwt);
      } else {
        this.setState({
          loading: false,
        });
        // alert('loadingnya' + loading);
      }
    } catch (error) {
      console.log('AsyncStorage Status Pinjam 2 Error: ' + error.message);
    }
  }

  async loadAbout() {
    try {
      const nAbout = 1;
      //if (value !== null) {
        this.props.getAbout(nAbout);
      //  console.log('about adalah : ' + this.props.getAbout.Caption);
      //} 
    } catch (error) {
      console.log('AsyncStorage Error: ' + error.message);
    }
  }

  // myNewTimer() {
  //   BackgroundTimer.runBackgroundTimer(() => { 
  //     //code that will be called every 3 seconds 
  //     console.log('tictoc');
  //     }, 
  //     1000);
  // }

  componentDidMount() {
    // AppState.addEventListener('change', this._handleAppStateChange);
    const {
      getNasabah,
      redMobile,
      getPinjaman,
      getPinjaman2,
      getPinjaman3,
      getVirtualAccount,
      delPinjaman,
      delPstatNull,
      getHisPinjaman,
      getAbout,
    } = this.props;

    // const gJWT= setTimeout(() => {
    //   this.loadJWT();
    // }, 5);
    const stsCek = setTimeout(() => {
      this.cekStatus();
    }, 500);
    const MyTimer = setTimeout(() => {
      this.pinjStatus();
    }, 1000);
    const MyTimer2 = setTimeout(() => {
      this.pinjStatus3();
    }, 2000);
    const MyTimer3 = setTimeout(() => {
      this.pinjHisStatus();
    }, 80);
    // const MyLogout = setTimeout(() => {
    //   this.logOutYn();
    // }, 10);

    // this.cekPembayaran();
    // this.props.getPinjaman3();
    // this.pinjStatus();
    // this.pinjStatus3();
    // this.pinjHisStatus();
    // this.props.getAbout(1);
  }

  // componentWillUnmount() {
  //   AppState.removeEventListener('change', this._handleAppStateChange);
  // }

  // _handleAppStateChange = nextAppState => {
  //   if (this.state.appState.match(/inactive|background/) && nextAppState === 'active') {
  //     console.log('App State: ' + 'App has come to the foreground!');
  //     alert('App State: ' + 'App has come to the foreground!');
  //   }
  //   console.log('App State: ' + nextAppState);
  //   alert('App State: ' + nextAppState);
  //   this.setState({ appState: nextAppState });
  //   if(nextAppState === 'background') { 
  //     this.myNewTimer();
  //   }else if(nextAppState === 'active'){
  //     // console.log('stopp coy');
  //     BackgroundTimer.stopBackgroundTimer();
  //     this.props.navigation.navigate('Login');
  //   }
  // };
  async logOutYn() {
      axios({
        method: 'Get',
        url: `http://103.121.149.77:63003/recbgyn`,
      })
        .then(res => {
        // console.log(res);
        const tmp_LogoutMain = res.data.datas.bgklik;
        // console.log('tmp_LogoutMain : ' + tmp_LogoutMain);
        this.setState({myLogoutMain: tmp_LogoutMain});
        // console.log(this.state.myLogoutMain);
          // if(this.state.myLogoutMain === 'y') {
          //   // console.log('uytuytuytuytuytu');
          //   this.stop_pinjStatus3;
          //   this.stop_pinjStatus;
          //   this.stop_ckStatus;
          // }
        })
        .catch(err => {
          this.setState({errCodeLogMain: err.response.status});
          console.log('error Log Out nich : ' + errCodeLogMain);
        });
  }

  async pinjStatus() {
    if (this.state.errCode === 400 || this.state.errCode === null) {
      axios({
        method: 'Get',
        url: `http://103.121.149.77:63003/pinjamk2/${this.state.jwt}`,
      })
        .then(res => {
          const statsPjm = res.data;
          // console.log(statsPjm);
          this.setState({statsPjm});
          this.setState({errCode: res.status});
          if (this.state.errCode === 200) {
            this.props.getPinjaman2(this.state.jwt);
            clearInterval(this.MyTimer);
          }
        })
        .catch(err => {
          this.setState({errCode: err.response.status});
          // console.log(this.state.errCode);
        });
      clearInterval(this.MyTimer);
    }
  }

  async pinjStatus3() {
    // this.setState({pinjLagi_Aktif: true});
    if (this.state.errCode3 === 400 || this.state.errCode3 === null) {
      axios({
        method: 'Get',
        url: `http://103.121.149.77:63003/pinjamk3/${this.state.jwt}`,
      })
        .then(res => {
          const statsPjm3 = res.data;

          this.setState({statsPjm3});
          this.setState({errCode3: res.status});
          if (this.state.errCode3 === 200) {
            this.setState({pinjLagi_Aktif: true});
            console.log('bener:' + this.state.errCode3);
            // let sTglPinjam = res;
            // console.log(res.data.datas);
            // clearInterval(this.MyTimer2);
            this.props.getPinjaman3(this.state.jwt);
            let sTglPinjam = res.data.datas.tglPinjam;
            let sTglPinjam_yr = sTglPinjam.substring(0, 4);
            let sTglPinjam_mm = sTglPinjam.substring(5, 7);
            let sTglPinjam_dd = sTglPinjam.substring(8, 10);
            let sTglPinjam_hh = sTglPinjam.substring(11, 13);
            let sTglPinjam_mmm = sTglPinjam.substring(14, 16);
            let sTglPinjam_ss = sTglPinjam.substring(17, 19);

            N_tglPinjam = sTglPinjam_yr + '-' + sTglPinjam_mm + '-' + sTglPinjam_dd + ' ' + sTglPinjam_hh + ':' + sTglPinjam_mmm + ':' + sTglPinjam_ss;

            let sJmlBayar = res.data.datas.jmlBayar;
            this.setState({N_jmlBayar: sJmlBayar});
            this.setState({N_tglPinjam: N_tglPinjam});
            this.setState({N_bunga: res.data.datas.bunga});
            this.setState({N_biaya_Admin: res.data.datas.biaya_admin});
            this.setState({N_tenorPinjam: res.data.datas.tenorPinjam});
            this.setState({N_tglJatuhTempo: res.data.datas.tglJatuhTempo});
            this.setState({N_jmlPinjam: res.data.datas.jmlPinjam});
            this.setState({N_tujuanPinjam: res.data.datas.tujuanPinjam});
            this.setState({N_tglBayar: res.data.datas.tglBayar});
            this.setState({N_status_pinjam: res.data.datas.status_pinjam});
            // console.log('tmpPinjm : ' + this.state.N_status_pinjam);
            // console.log('jmlBayar: ' + this.state.N_jmlBayar);
           // this.setState({rePinjam: 0});
            this.props.delPinjaman(this.state.jwt);
            axios({
              method: 'post',
              // url: `http://192.168.5.27:63003/pinjam/${this.state.jwt}`,
              url: `http://103.121.149.77:63003/pinjamhis/${this.state.jwt}`,
              headers: {},
              data: {
                jmlPinjam: this.state.N_jmlPinjam,
                tujuanPinjam: this.state.N_tujuanPinjam,
                tglBayar: this.state.N_tglBayar,
                status_pinjam: '5',
                pinjaman_dt: this.state.N_tglPinjam,
                tglJatuhTempo: this.state.N_tglJatuhTempo,
                keterangan: 'Lunas',
                jmlBayar: this.state.N_jmlBayar,
                bunga: this.state.N_bunga,
                biaya_admin: this.state.N_biaya_Admin,
                tenorPinjam: this.state.N_tenorPinjam,
              },
            })
              .then(res => {
                console.log(res);
                // this.pinjHisStatus();
              })
              .catch(err => {
                //handle error
                console.log(err);
              });
            // this.props.delPinjaman(this.state.jwt);
          }
        })
        .catch(err => {
          //handle error
          // errCode = err.response.status;
          this.setState({errCode3: err.response.status});
          // console.log('errCode3 salah' + this.state.errCode3);
        });
      clearInterval(this.MyTimer2);
      // this.stopInterval();
      // this.stop_pinjStatus3();
    }
  }

  stop_ckStatus = () => {
    clearInterval(this.stsCek);
  };

  stop_pinjStatus3 = () => {
    clearInterval(this.MyTimer2);
  };

  stop_pinjStatus = () => {
    clearInterval(this.MyTimer);
  };

  async pinjHisStatus() {
    if (this.state.hisErrCode === 400 || this.state.hisErrCode === null) {
      axios({
        method: 'Get',
        url: `http://103.121.149.77:63003/pinjamhis/${this.state.jwt}`,
      })
        .then(res => {
          const hisPinjm = res.data;
          // console.log(res.status);
          //console.log(res.data.datas);
          // alert(res.data.datas.length);
          if (res.data.datas.length > 0) {
            this.setState({hisErrCode: res.status});
            clearInterval(this.MyTimer3);
            if (this.state.hisErrCode === 200) {
              this.setState({historyExist: 1});
              //console.log('ada history ' + this.state.historyExist);
              // this.stop_pinjStatus3();
              // this.stopInterval();
              if (this.state.historyExist === 1) {
                this.props.delPinjaman(this.state.jwt);
              }
              // this.setState({errCode3: 400});
              // this.setState({errCode: 400});
            }
          } else {
            this.setState({hisErrCode: 400});
            // this.setState({historyExist: 0});
            //console.log('tidak ada history ' + this.state.historyExist);
            clearInterval(this.MyTimer3);
          }
        })
        .catch(err => {
          //handle error
          // errCode = err.response.status;
          this.setState({hisErrCode: err.reponse.status});
          // console.log('errCode3 salah' + this.state.errCode3);
        });
      clearInterval(this.MyTimer3);
    }
  }

  async cekStatus() {
    this.props.getNasabah(this.state.jwt);
    if(this.prop.nasabah.verified_status !== null || this.prop.nasabah.verified_status === 1){
      let looptime = 3000;
      const timerHandle = setTimeout(() => {
        if(this.props.nasabah.verified_status === 1)
      {
          {this.props.getNasabah(this.state.jwt)}
      }
          }, looptime);
      }
  }

  async _Ajukan() {
    // eslint-disable-next-line no-lone-blocks
    await this.loadJWT();
    {
      !this.props.nasabah
        ? this.props.navigation.navigate('Registration')
        : this.props.nasabah.verified_status === 1
        ? alert('Menunggu Konfirmasi')
        : alert('Proses Pengajuan anda lagi diproses');
    }
  }

  stopInterval = () => {
    clearInterval(this.MyTimer3);
  };

  ajukanPinjamanLagi() {
    this.setState({rePinjam: 1});
  }

  async cekPembayaran() {
    let myInterval = setInterval(() => {
      if (this.state.hitungan >= 2) {
        clearInterval(myInterval);
      } else {
        this.setState({hitungan: this.state.hitungan + 1});
        const {pinjaman} = this.props;

        this.props.getPinjaman3(this.state.jwt);
        //console.log('hitungan' + this.state.hitungan );
        //console.log('nomer telepon' + this.state.jwt );
        // console.log('pembayaran : ' + pinjaman.status_pinjam);

        //if (this.state.hitungan === 2) {
          // console.log('Nomor ID Bank: ' + virtual.id);
          // console.log('pembayaran : ' + pinjaman.status_pinjam);
       // }
      }
    }, 10000);
  }

  apply_pinjaman() {
    // eslint-disable-next-line no-lone-blocks
    deviceStorage.saveStatus('myStatus', 2);
    {
      //this.setState({rePinjam: 2});
      this.props.delPstatNull(this.state.jwt);

      !this.state.besarPinjam ||
      !this.state.tujuanPinjam ||
      !this.state.tanggalPinjam
        ? alert('Silahkan Pilih Tujuan dan Tanggal Bayar')
        : axios({
            method: 'post',
            // url: `http://192.168.5.27:63003/pinjam/${this.state.jwt}`,
            url: `http://103.121.149.77:63003/pinjam/${this.state.jwt}`,
            headers: {},
            data: {
              jmlPinjam: this.state.besarPinjam,
              tujuanPinjam: this.state.tujuanPinjam,
              tglBayar: this.state.tanggalPinjam,
              status_pinjam: '1',
            },
          })
            .then(res => {
              // this.props.getNasabah(this.state.jwt);
              // alert(res.data.message);
              // console.log(res);

              this.props.getPinjaman(this.state.jwt);
              alert(res.data.message);
              //console.log(res);
            })
            .catch(err => {
              //handle error
              console.log(err);
            });
    }
  }

  lengkapi() {
    this.props.navigation.navigate('Registration');
  }

  kirimVA() {
  // kirimVA = () => {
    this.setState({btnBayar: !this.state.btnBayar});
    // this.state.btnBayar ? require('/commons/Btn');
    
    // this.setState(previousState => ({btnBayar: !previousState.btnBayar}));
    axios({
      method: 'Get',
      // url: `http://103.121.149.77:63003/virtualaccount/${this.state.bnkname}`,
      url: `http://103.121.149.77:63003/virtualaccount/${this.props.nasabah.id_bank}`,
    })
      .then(res => {
        const statVirtual = res.data.datas;
        //  console.log(statVirtual.no_va);
        // console.log(res.status);

        this.setState({statVirtual: statVirtual.no_va});
        //  console.log('statVirtual : ' + statVirtual.no_va);

        this.setState({errCode: res.status});
        // console.log(this.state.errCode)
      })
      .catch(err => {
        //handle error
        // console.log(err.response.status);
        console.log(err);
        // errCode = err.response.status;
        // this.setState({errCode: res.status})
      });
  }

  render() {
    const {nasabah, pinjaman, pinjaman2, pinjaman3, virtual, about, NasErr} = this.props;
    let virtualNew = this.state.statVirtual;

    // { NasErr === 'network error' ? (
    //   <Image source={IMAGE.noConnection} />
    // ) : NasErr !== 'network error' ? (
      return (
        //console.log('NasErr 2: ' + this.props.NasErr),
        //console.log('nasabah 2: ' + this.props.nasabah.noktp),
        console.log('this.state.historyExist : ' + this.state.historyExist),
    
        <View>
          {/* <Spinner visible={this.props.loadings} textContent={'Loading...'} /> */}
          <Header title="Home" isHome={true} navigation={this.props.navigation} />
          <View style={styles.Container}>
          {/* { NasErr === 'network error' ? (
            <Image source={IMAGE.noConnection} />
          ) :  NasErr === null ? ( */}
            <>

            {this.state.historyExist === 1 && this.state.rePinjam === 0 ? (
                <View style={styles.Lunas}>
                  <Image source={IMAGE.PaymentDone} />
                  <Text style={styles.LunasB1}>Anda Sudah Melunasi tagihan</Text>
                  <Text>Terimakasih Sudah mengunakan Jasa kami</Text>                  
                </View>
              ): this.state.historyExist === 1 && this.state.rePinjam === 1 ? (
                <MainFresh
                  handlerChangeValue={this.handlerChangeValue}
                  navigation={this.props.navigation}
                />
              ):     
             nasabah === null && this.state.historyExist === null ? (
              //  console.log('NasErr 3n: ' + NasErr),
              //console.log('nasabah 3: ' + nasabah.verified_status),
                <Image
                  source={this.state.img}
                  style={{width: 256, height: 300}}
                />
              ) : nasabah && nasabah.verified_status === 1 ? (
                <Image
                  source={this.state.imgWait}
                  style={{width: 256, height:250}}
                />
              ) : nasabah && nasabah.verified_status === 2 && pinjaman  === null && this.state.rePinjam === 0 ? (
                  <MainFresh
                    handlerChangeValue={this.handlerChangeValue}
                    navigation={this.props.navigation}
                  />
              ) : nasabah &&
                nasabah.verified_status === 2 &&
                pinjaman &&
                pinjaman.status_pinjam === '1' &&
                this.state.rePinjam === 2  ? (
                  <Image
                  source={this.state.usrWait}
                  style={{width: 268, height:250}}
                />
              ) : nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '2' ? (
                <MainFresh2
                  handlerChangeValue={this.handlerChangeValue}
                  navigation={this.props.navigation}
                />
              ) : null}

            {nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '2' &&
              virtualNew !== 'null' ? (
                <FormVirtualAcc
                  handlerChangeValue={this.handlerChangeValue}
                  navigation={this.props.navigation}
                />
              ) : null}

              {/* {nasabah &&
              nasabah.verified_status === 2 &&
              this.state.historyExist === 1 &&
              this.state.errCode3 === 400 &&
              this.state.errCode === 400 &&

              this.state.rePinjam === 0 &&
              this.state.hisErrCode === 200 &&
              pinjaman &&
              pinjaman.status_pinjam !== '1' ? (
                <View style={styles.Lunas}>
                  <Image source={IMAGE.PaymentDone} />
                  <Text style={styles.LunasB1}>Anda Sudah Melunasi tagihan</Text>
                  <Text>Terimakasih Sudah mengunakan Jasa kami</Text>                  
                </View>
              ) : null} */}

              {/* {nasabah &&
              nasabah.verified_status === 2 &&
              this.state.errCode3 === 200 &&
              this.state.rePinjam === 0 &&
              pinjaman &&
              pinjaman.status_pinjam !== '1' ? (
                <View style={styles.Lunas}>
                  <Image source={IMAGE.PaymentDone} />
                  <Text style={styles.LunasB1}>Anda Sudah Melunasi tagihan</Text>
                  <Text>Terimakasih Sudah mengunakan Jasa kami</Text>
                </View>
              ) : null} */}

              {/* {nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '5' ? (
                <View style={styles.Lunas}>
                  <Image source={IMAGE.PaymentDone} />
                  <Text style={styles.LunasB1}>Anda Sudah Melunasi tagihan</Text>
                  <Text>Terimakasih Sudah mengunakan Jasa kami</Text>
                </View>
              ) : null} */}

              {/* {nasabah &&
              nasabah.verified_status === 2 &&
              this.state.errCode3 === 400 &&
              this.state.errCode === 400 &&
              this.state.hisErrCode === 400 &&
              pinjaman === null &&
              virtualNew === 'null' ? (
                <>
                  <MainFresh
                    handlerChangeValue={this.handlerChangeValue}
                    navigation={this.props.navigation}
                  />
                </>
              ) : nasabah &&
              nasabah.verified_status === 2 &&
              this.state.rePinjam === 1 ? (
                <>
                  <MainFresh
                    handlerChangeValue={this.handlerChangeValue}
                    navigation={this.props.navigation}
                  />
                </>
              ) : null} */}

              {/* {nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '1' ? (
                <Image
                  source={this.state.usrWait}
                  style={{width: 268, height:250}}
                />
              ) : null}

              {nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '2' &&
              virtualNew === 'null' ? (
                <MainFresh2
                  handlerChangeValue={this.handlerChangeValue}
                  navigation={this.props.navigation}
                />
              ) : null}

              {nasabah &&
              nasabah.verified_status === 2 &&
              pinjaman &&
              pinjaman.status_pinjam === '2' &&
              virtualNew !== 'null' ? (
                <FormVirtualAcc
                  handlerChangeValue={this.handlerChangeValue}
                  navigation={this.props.navigation}
                />
              ) : null} */}

              <View style={{width: '90%', paddingTop: 20}}>
              {this.state.historyExist === 1 && this.state.rePinjam === 0 ? (
                     <Btn
                     title="Ajukan Pinjaman Lagi"
                     onPress={() => this.ajukanPinjamanLagi()} 
                     />
              ): this.state.historyExist === 1 && this.state.rePinjam === 1 &&
                  nasabah &&
                  nasabah.verified_status === 2 &&
                  pinjaman  === null ? (
                  <Btn
                    title="AJUKAN PINJAMAN"
                    onPress={() => this.apply_pinjaman()}
                    />
              ): nasabah && nasabah === null && this.state.historyExist === null ? (
                  <Btn title="LENGKAPI DATA" onPress={() => this._Ajukan()} />
                ) : nasabah && nasabah.verified_status === 1 ? (
                  <Btn title="MENUNGGU KONFIRMASI DATA" />
                
                // ) : nasabah &&
                //   nasabah.verified_status === 2 &&
                //   this.state.errCode3 === 400 &&
                //   this.state.errCode === 400 &&
                //   this.state.hisErrCode === 400 &&
                //   pinjaman === null ? (
                //   <Btn
                //     title="AJUKAN PINJAMAN"
                //     onPress={() => this.apply_pinjaman()}
                //   />
                // ) : nasabah &&
                //   nasabah.verified_status === 2 &&
                //   this.state.rePinjam === 1 ? (
                //   <Btn
                //     title="AJUKAN PINJAMAN"
                //     onPress={() => this.apply_pinjaman()}
                //   /> 
                ) : nasabah &&
                  nasabah.verified_status === 2 &&
                  pinjaman &&
                  pinjaman.status_pinjam === '1' ? (
                  <Btn title="PINJAMAN DIPROSES" />
                ) : nasabah &&
                  nasabah.verified_status === 2 &&
                  pinjaman &&
                  pinjaman.status_pinjam === '2' && 
                  this.state.btnBayar === false ? (
                  <Btn title="Bayar Sekarang" onPress={() => this.kirimVA()} />
                // ) : nasabah &&
                //   nasabah.verified_status === 2 && 
                //   this.state.historyExist === 1  &&
                //   this.state.errCode3 === 400 &&
                //   this.state.errCode === 400 &&
                //   this.state.hisErrCode === 200 &&
                //   this.state.rePinjam === 0 &&
                //   this.state.pinjLagi_Aktif === true &&
                //   pinjaman &&
                //   pinjaman.status_pinjam !== '1' ? (
                //     this.setState({pinjLagi_Aktif: !this.state.pinjLagi_Aktif}),
                //     // console.log('pinjLagi_aktif : ' + !this.state.pinjLagi_Aktif ),
                //     <Btn
                //     title="Ajukan Pinjaman Lagi"
                //     onPress={() => this.ajukanPinjamanLagi()}
                //   />
                ) : nasabah &&
                  nasabah.verified_status === 2 &&
                  pinjaman  === null ? (
                   <Btn
                    title="AJUKAN PINJAMAN"
                    onPress={() => this.apply_pinjaman()}
                    />
                ) : null}
                

                {/* {nasabah &&
                nasabah.verified_status === 2 && 
                this.state.historyExist === 1  &&
                this.state.errCode3 === 400 &&
                this.state.errCode === 400 &&
                this.state.hisErrCode === 200 &&
                this.state.rePinjam === 0 &&
                this.state.pinjLagi_Aktif === false &&
                pinjaman &&
                pinjaman.status_pinjam !== '1'  ? (
                  // this.pinjHisStatus(),
                  <Btn
                    title="Ajukan Pinjaman Lagi"
                    onPress={() => this.ajukanPinjamanLagi()}
                  /> ) 
                  : null} */}
              </View>
            </>
          </View>
        </View>
      );
    
  }
}

const styles = StyleSheet.create({
  Container: {
    backgroundColor: '#bdd9e2',
    borderRadius: 10,
    marginHorizontal: 20,
    padding: 10,
    position: 'absolute',
    top: 50,
    left: 0,
    right: 0,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,

    elevation: 5,
  },
  byrBtn: {
    top: 280,
    width: '95%',
  },
  Lunas: {
    width: '90%',
    paddingTop: 10,
    justifyContent: 'center',
    alignItems: 'center',
  },
  LunasB1: {
    fontSize: 18,
    fontWeight: 'bold',
  },
});

// export default MainScreen;

const mapStoreToProps = state => ({
  loadings: state.nasabah.loading,
  nasabah: state.nasabah.nasabah,
  redMobile: state.register.redMobile,
  pinjaman: state.pinjaman.pinjaman,
  // pinjaman2: state.pinjaman.pinjaman,
  // pinjaman3: state.pinjaman.pinjaman,
  // myVirtual: state.virtual.myVirtual,
  virtual: state.virtual.virtual,
  about: state.about.about,
  NasErr: state.nasabah.error,
});
const mapDispatchToProps = dispatch => ({
  getNasabah: mobile => dispatch(getNasabah(mobile)),
  getPinjaman: mobile => dispatch(getPinjaman(mobile)),
  getPinjaman2: mobile => dispatch(getPinjaman2(mobile)),
  getPinjaman3: mobile => dispatch(getPinjaman3(mobile)),
  getHisPinjaman: mobile => dispatch(getHisPinjaman(mobile)),
  delPinjaman: mobile => dispatch(delPinjaman(mobile)),
  delPstatNull: mobile => dispatch(delPstatNull(mobile)),
  // getVirtualAcc: bankname => dispatch(getVirtualAcc(bankname)),
  getVirtualAccount: bankname => dispatch(getVirtualAccount(bankname)),
  setLoading: p => dispatch(setLoading(p)),
  getAbout: id => dispatch(getAbout(id)),
});

export default connect(
  mapStoreToProps,
  mapDispatchToProps,
)(MainScreen);
