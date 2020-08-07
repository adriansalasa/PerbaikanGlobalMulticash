/* eslint-disable react-native/no-inline-styles */
import React, {Component} from 'react';
import {
  View,
  Text,
  StyleSheet,
  TextInput,
  Image,
  Clipboard,
  Button,
} from 'react-native';
import SnapSlider from 'react-native-snap-slider';
import ModalDropdown from 'react-native-modal-dropdown';
import {Btn, Img} from '../commons/';
import {getVirtualAccount} from '../../redux/action/VirtualAction';
import {getNasabah, setLoading} from '../../redux/action/NasabahAction';
import {getPinjaman} from '../../redux/action/PinjamanAction';
import {connect} from 'react-redux';
import NumberFormat from 'react-number-format';
import {
  ContentStyleDetail,
  ContentStyleDetail2,
  ContentStyleDetail3,
  ContentStyleDetail4,
} from '../../components/tabs/utama/MainStyle';
import {ContentWrapper} from '../../components/tabs/utama/MainStyle';
import AsyncStorage from '@react-native-community/async-storage';
// import { Clipboard } from 'react-native';

class FormVirtualAcc extends Component {
  constructor(props) {
    super(props);

    this.state = {
      jwt: null,
      // pathGambar: '',
      // img: require('../../assets/images/BCA_Logo.png'),
      // img: require(pathGambar/namaGambar),
      hitungan: 0,
      AccVirtual: 0,
      AccVirtual1: 0,
      AccVirtual2: 0,
      AccVirtual3: 0,
      AccVirtual4: 0,
      strAccVirtual: '',
      strJmlPnjm: '',
      clipboardContent: null,
      bunga: 0,
      tenorPinjam: 0,
      biaya_admin: 0,
      TotalBayar: 0,
    };
    // this.loadJWT = this.loadJWT.bind(this);
    this.loadJWT();
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
      } else {
        this.setState({
          loading: false,
        });
      }
    } catch (error) {
      console.log('AsyncStorage Error: ' + error.message);
    }
  }

  async ckAccount() {
    let myInterval = setInterval(() => {
      if (this.state.hitungan >= 2) {
        clearInterval(myInterval);
        //  this.setState({hitungan: this.state.hitungan+1});
      } else {
        this.setState({hitungan: this.state.hitungan + 1});
        const {virtual, nasabah, pinjaman} = this.props;

        this.props.getVirtualAccount(nasabah.id_bank);
        // console.log('hitungan' + this.state.hitungan )

        if (this.state.hitungan === 2) {
          // console.log('Nomor ID Bank: ' + virtual.id);
          this.setState({AccVirtual: virtual.no_va});
          this.setState({strAccVirtual: virtual.no_va});
          this.setState({strJmlPnjm: pinjaman.jmlPinjam});
          console.log(this.state.AccVirtual);
          console.log(this.state.strJmlPnjm);

          let AccVirtual1 = this.state.AccVirtual.substring(0, 4);
          let AccVirtual2 = this.state.AccVirtual.substring(4, 8);
          let AccVirtual3 = this.state.AccVirtual.substring(8, 12);
          let AccVirtual4 = this.state.AccVirtual.substring(12, 16);
          // console.log(AccVirtual1);
          // console.log(AccVirtual2);
          // console.log(AccVirtual3);
          // console.log(AccVirtual4);
          this.setState({AccVirtual1: AccVirtual1});
          this.setState({AccVirtual2: AccVirtual2});
          this.setState({AccVirtual3: AccVirtual3});
          this.setState({AccVirtual4: AccVirtual4});
          this.setState({bunga: this.state.bunga + 50000});
          this.setState({biaya_admin: this.state.biaya_admin + 10000});
          this.setState({tenorPinjam: this.state.tenorPinjam + 30});
          this.setState({
            TotalBayar:
              parseInt(pinjaman.jmlPinjam) +
              parseInt(this.state.bunga) +
              parseInt(this.state.biaya_admin),
              // parseInt(this.state.tenorPinjam),
          });
          // let pGambar = this.state.pathGambar + nasabah.id_bank + '.png';
          // this.setState({pathGambar : '../../assets/images/' + pGambar })
          // pathGambar:require('../../assets/images/009.png' );
          // console.log('path gambar' + this.state.pathGambar)
          console.log('totalBayar' + this.state.TotalBayar);
        }
      }
    }, 500);
  }

  componentDidMount() {
    this.ckAccount();
  }

  // readFromClipboard = async () => {
  //   const clipboardContent = await Clipboard.getString();
  //   this.setState({clipboardContent});
  // };

  pasteClipboardPinjaman = async () => {
    await Clipboard.setString(this.state.strJmlPnjm);
    alert('Jumlah bayar Copied!');
  };

  pasteClipboardVA = async () => {
   await Clipboard.setString(this.state.AccVirtual);
    alert('Virtual Account Copied!2');
  };

  render() {
    const {pinjaman, virtual, nasabah} = this.props;

    return (
      <>
        {pinjaman && pinjaman.status_pinjam === '2' ? (
          <>
            <View style={ContentStyleDetail}>
              <View style={styles.OptionsBaris}>
                <View style={styles.Label}>
                  <Text style={styles.labelTextRightKD}>Kode bank</Text>
                </View>
              </View>

              <View style={styles.OptionsBarisJdl}>
                <View style={styles.Label}>
                  {nasabah.id_bank === '014' ? (
                    <Image
                      source={Img.ICON_BCA}
                      style={{width: 120, height: 38}}
                    />
                  ) : nasabah.id_bank === '009' ? (
                    <Image
                      source={Img.ICON_BNI}
                      style={{width: 110, height: 38}}
                    />
                  ) : nasabah.id_bank === '008' ? (
                    <Image
                      source={Img.ICON_MANDIRI}
                      style={{width: 150, height: 40}}
                    />
                  ) : nasabah.id_bank === '002' ? (
                    <Image
                      source={Img.ICON_BRI}
                      style={{width: 40, height: 40}}
                    />
                  ) : (
                    <Image
                      source={Img.ICON_PERMATA}
                      style={{width: 165, height: 35}}
                    />
                  )}
                </View>

                <View style={styles.Label}>
                  <Text style={styles.labelTextRight}>{nasabah.id_bank}</Text>
                </View>
              </View>

              <View style={styles.lineStyleAtas} />

              <View style={styles.OptionsBarisPNR}>
                <View style={styles.Label}>
                  <Text style={styles.labelText}>Nama penerima</Text>
                </View>

                <View style={styles.Label}>
                  <Text style={styles.labelTextRightSub}>Globalmulticash</Text>
                </View>
              </View>
              <Text style={styles.OptionsPinjamJdl}>
                Jumlah Pembayaran (Rp)
              </Text>
              <Text style={styles.Currency}>
                <NumberFormat
                  value={this.state.TotalBayar}
                  displayType={'text'}
                  thousandSeparator={true}
                  prefix={'Rp '}
                  renderText={value => <Text>{value}</Text>}
                />
              </Text>
              {/* <Text style={styles.jdlRed}>
                Mohon bayar sesuai jumlah yg tertera
              </Text> */}
              {/* <View style={styles.copyVA_Kosong} /> */}

              <View style={styles.lineStyleBawah} />
              <View style={{bottom: 5}}><Button
                onPress={this.pasteClipboardPinjaman}
                title="salin jumlah bayar"
              /></View>

              <View style={styles.lblVA}>
                {/* <Text style={styles.labelText}>Rekening Virtual Anda</Text> */}
              </View>
              {/* <View style={styles.OptionsBaris}> */}
                {/* <View style={styles.Label}> */}
                  <Text style={styles.textInput}>{this.state.AccVirtual1}-{this.state.AccVirtual2}
                  -{this.state.AccVirtual3}-{this.state.AccVirtual4}</Text>
                {/* </View> */}

                {/* <View style={styles.Label}>
                  <Text style={styles.textInput}>{this.state.AccVirtual2}</Text>
                </View>

                <View style={styles.Label}>
                  <Text style={styles.textInput}>{this.state.AccVirtual3}</Text>
                </View>

                <View style={styles.Label}>
                  <Text style={styles.textInput}>{this.state.AccVirtual4}</Text>
                </View> */}
              {/* </View> */}
              
              
              {/* <View style={styles.OptionsBaris}>
                <Text
                  style={styles.copyVA_Input}
                  onPress={Clipboard.setString(this.state.strAccVirtual)}>
                  Salin rekening virtual anda
                </Text>
              </View> */}
              <View style={styles.lineStyleAkhir} />
            </View>
            <View style={{marginTop: 50}}>
            <Button
                onPress={this.pasteClipboardVA}
                title="salin rekening virtual anda"
              />
              </View>
            {/* <View style={styles.BtnjmlPinjm}>
              <Button
                onPress={this.pasteClipboardPinjaman}
                title="salin jumlah bayar"
              />
            </View>

            <View style={styles.BtnVA}>
              <Button
                onPress={this.pasteClipboardVA}
                title="salin rekening virtual anda"
              />
            </View> */}
          </>
        ) : null}
      </>
    );
  }
}

const styles = StyleSheet.create({
  jmlByr: {
    color: 'white',
    fontSize: 11,
    marginLeft: 10,
    fontWeight: 'bold',
    marginTop: 5,
  },

  jdlRed: {
    color: 'red',
    fontSize: 11,
    marginLeft: 10,
    fontWeight: 'bold',
    marginTop: 5,
  },
  lineStyleAtas: {
    // color: 'black',
    // // textDecorationStyle: 'dotted',
    // fontSize: 12,
    marginTop: 15,
    borderStyle: 'dashed',
    borderWidth: 2,
    borderRadius: 1,
    borderColor: '#eaeaea',
    width: '100%',
  },
  lineStyleBawah: {
    // color: 'black',
    // // textDecorationStyle: 'dotted',
    // fontSize: 12,
    top: 45,
    marginTop: 15,
    borderStyle: 'dashed',
    borderWidth: 2,
    borderRadius: 1,
    borderColor: '#eaeaea',
    width: '100%',
  },
  lineStyleAkhir: {
    // color: 'black',
    // // textDecorationStyle: 'dotted',
    // fontSize: 12,
    top: 20,
    marginTop: 60,
    width: '100%',
  },
  Currency: {
    marginTop: 10,
    fontSize: 25,
    color: 'black',
    fontWeight: 'bold',
  },
  LilCurrency: {
    fontSize: 14,
    color: 'grey',
    fontWeight: 'bold',
    marginHorizontal: 25,
  },
  textTgl: {
    color: 'grey',
    fontWeight: 'bold',
    marginHorizontal: 25,
  },
  dropdownStyle: {
    width: 160,
    borderRadius: 3,
    fontSize: 18,
    alignItems: 'flex-end',
    paddingRight: 20,
  },
  dropdownText: {
    fontSize: 14,
    color: 'black',
    textAlign: 'center',
    textAlignVertical: 'center',
  },
  OptionsPinjam: {flexDirection: 'row', marginTop: 20},
  OptionsBaris: {flexDirection: 'row', marginTop: 10},
  lblVA: {flexDirection: 'row', marginTop: 15},
  BtnjmlPinjm: {flexDirection: 'row', marginTop: 250},
  BtnVA: {flexDirection: 'row', marginTop: 120, top: 5},
  OptionsBarisPNR: {flexDirection: 'row', marginTop: 20},
  OptionsBarisJdl: {flexDirection: 'row'},
  OptionsPeriod: {flexDirection: 'row', marginTop: 15, marginBottom: 10},
  OptionsPinjamJdl: {
    flexDirection: 'row',
    marginTop: 30,
    marginLeft: 10,
    color: 'grey',
  },

  Label: {
    flex: 1,
    justifyContent: 'center',
    marginHorizontal: 6,
    left: 3,
  },
  LabelPnr: {
    flex: 1,
    justifyContent: 'center',
    marginHorizontal: 6,
    left: 5,
  },
  LabelPeriod: {
    flex: 1,
    justifyContent: 'center',
    marginHorizontal: 15,
    color: '#3e3e3e',
    fontWeight: 'bold',
  },
  dropdownDropdown: {
    width: 150,
    height: 300,
    borderWidth: 2,
    borderRadius: 3,
  },
  labelSub: {
    color: '#3e3e3e',
    fontWeight: 'bold',
  },
  labelSubBayar: {
    fontSize: 30,
    color: 'black',
    fontWeight: 'bold',
  },
  labelText: {
    color: 'grey',
    fontWeight: 'bold',
  },
  labelTextTitle: {
    color: 'black',
    fontSize: 22,
    fontWeight: 'bold',
  },
  labelTextRightKD: {
    color: 'grey',
    fontSize: 10,
    marginLeft: 240,
  },
  labelTextRight: {
    color: 'black',
    fontWeight: 'bold',
    marginLeft: 110,
  },
  labelTextRightSub: {
    color: 'grey',
    fontWeight: 'bold',
    marginLeft: 35,
  },
  labelTextPeriod: {
    color: '#3e3e3e',
    fontWeight: 'bold',
    marginRight: 3,
  },
  labelHeadTitle: {
    color: 'grey',
    fontWeight: '800',
    textAlign: 'center',
    marginLeft: 80,
  },
  textInput: {
    // height: 50,
    backgroundColor: 'white',
    marginTop: 4,
    marginBottom: 4,
    borderWidth: 1,
    borderColor: 'green',
    borderRadius: 5,
    padding: 4,
    fontSize: 16,
    fontWeight: 'bold',
  },
  bluetextInput: {
    backgroundColor: 'lightskyblue',
    marginTop: 8,
    marginBottom: 16,
    borderWidth: 1,
    borderColor: 'white',
    borderRadius: 5,
    padding: 5,
    fontSize: 12,
    fontWeight: 'bold',
    color: 'white',
    opacity: 80,
  },

  copyVA_Input: {
    backgroundColor: 'lightskyblue',

    marginBottom: 5,
    borderWidth: 1,
    borderColor: 'white',
    borderRadius: 5,
    padding: 5,
    fontSize: 12,
    fontWeight: 'bold',
    color: 'white',
    opacity: 80,
  },
  copyVA_Kosong: {
    marginBottom: 5,
    borderWidth: 1,
    borderColor: 'white',
    borderRadius: 5,
    padding: 5,
    fontSize: 12,
    fontWeight: 'bold',
    color: 'white',
    opacity: 80,
  },
  snapsliderContainer: {marginTop: 10},
  snapslider: {},
  snapsliderItemWrapper: {},
  snapsliderItem: {},
});

const mapStoreToProps = state => ({
  redMobile: state.register.redMobile,
  loading: state.opsi.loading,
  pinjaman: state.pinjaman.pinjaman,
  nasabah: state.nasabah.nasabah,
  virtual: state.virtual.virtual,
});
const mapDispatchToProps = dispatch => ({
  getPinjaman: mobile => dispatch(getPinjaman(mobile)),
  getVirtualAccount: bankname => dispatch(getVirtualAccount(bankname)),
  getNasabah: mobile => dispatch(getNasabah(mobile)),
});

export default connect(
  mapStoreToProps,
  mapDispatchToProps,
)(FormVirtualAcc);
