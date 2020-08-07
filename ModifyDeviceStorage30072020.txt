import AsyncStorage from '@react-native-community/async-storage';

const deviceStorage = {
  async saveKey(key, valueToSave) {
    try {
      await AsyncStorage.setItem(key, valueToSave);
    } catch (error) {
      console.log('AsyncStorage Error: ' + error.message);
    }
  },

  async saveStatus(key, valueToSave) {
    try{
      await AsyncStorage.setItem(key, JSON.stringify(valueToSave));
      console.log('value save status : ' + valueToSave);
    }catch(error) {
      console.log('error Set status : ' + error.message);
    }
  },

  async loadMyStatus() {
    try{
      const myValue = await AsyncStorage.getItem('myStatus');
      if(myValue !== null) {
        this.setState({
          myStat: myValue
        });
        console.log('Load status : ' + myValue);
      }
    }catch(error) {
      console.log('AsyncStorage My Status Error: ' + error.message);
    }
  },

  async loadJWT() {
    try {
      const value = await AsyncStorage.getItem('id_token');
      if (value !== null) {
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
  },

  async deleteJWT() {
    try {
      await AsyncStorage.removeItem('id_token').then(() => {
        this.setState({
          jwt: '',
        });
      });
    } catch (error) {
      console.log('AsyncStorage Error: ' + error.message);
    }
  },

};

export default deviceStorage;
