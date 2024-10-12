# converter
import React, { useState, useEffect } from 'react';
import {
  StyleSheet,
  Text,
  View,
  TextInput,
  Button,
  Picker,
  FlatList,
  TouchableOpacity,
  AsyncStorage,
} from 'react-native';

// API for fetching real-time exchange rates
const API_URL = 'https://api.exchangeratesapi.io/latest?base=';

export default function App() {
  const [amount, setAmount] = useState('');
  const [fromCurrency, setFromCurrency] = useState('USD');
  const [toCurrency, setToCurrency] = useState('EUR');
  const [convertedAmount, setConvertedAmount] = useState(null);
  const [exchangeRate, setExchangeRate] = useState(null);
  const [favorites, setFavorites] = useState([]);

  // Fetch exchange rate when currencies are selected
  useEffect(() => {
    if (fromCurrency && toCurrency) {
      fetchExchangeRate();
    }
  }, [fromCurrency, toCurrency]);

  // Fetch exchange rate from the API
  const fetchExchangeRate = async () => {
    try {
      const response = await fetch(`${API_URL}${fromCurrency}`);
      const data = await response.json();
      setExchangeRate(data.rates[toCurrency]);
    } catch (error) {
      alert('Error fetching exchange rates.');
    }
  };

  // Convert the entered amount
  const convertCurrency = () => {
    const amt = parseFloat(amount);
    if (!isNaN(amt) && exchangeRate) {
      const result = amt * exchangeRate;
      setConvertedAmount(result.toFixed(2));
    } else {
      alert('Please enter a valid amount.');
    }
  };

  // Save favorite currency pair to AsyncStorage
  const saveFavorite = async () => {
    const pair = `${fromCurrency} to ${toCurrency}`;
    try {
      const currentFavorites = await AsyncStorage.getItem('favorites');
      const favoritesArray = currentFavorites ? JSON.parse(currentFavorites) : [];
      if (!favoritesArray.includes(pair)) {
        favoritesArray.push(pair);
        await AsyncStorage.setItem('favorites', JSON.stringify(favoritesArray));
        setFavorites(favoritesArray);
      }
    } catch (error) {
      alert('Error saving favorite pair.');
    }
  };

  // Load favorite currency pairs from AsyncStorage
  const loadFavorites = async () => {
    try {
      const savedFavorites = await AsyncStorage.getItem('favorites');
      if (savedFavorites) {
        setFavorites(JSON.parse(savedFavorites));
      }
    } catch (error) {
      alert('Error loading favorite pairs.');
    }
  };

  // Remove a favorite currency pair from AsyncStorage
  const removeFavorite = async (pair) => {
    try {
      const currentFavorites = await AsyncStorage.getItem('favorites');
      const favoritesArray = currentFavorites ? JSON.parse(currentFavorites) : [];
      const updatedFavorites = favoritesArray.filter((item) => item !== pair);
      await AsyncStorage.setItem('favorites', JSON.stringify(updatedFavorites));
      setFavorites(updatedFavorites);
    } catch (error) {
      alert('Error removing favorite pair.');
    }
  };

  useEffect(() => {
    loadFavorites();
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Currency Converter</Text>

      <TextInput
        style={styles.input}
        placeholder="Enter amount"
        keyboardType="numeric"
        value={amount}
        onChangeText={(value) => setAmount(value)}
      />

      <View style={styles.pickerContainer}>
        <Text style={styles.label}>From:</Text>
        <Picker
          selectedValue={fromCurrency}
          style={styles.picker}
          onValueChange={(value) => setFromCurrency(value)}
        >
          <Picker.Item label="USD" value="USD" />
          <Picker.Item label="EUR" value="EUR" />
          <Picker.Item label="GBP" value="GBP" />
          <Picker.Item label="JPY" value="JPY" />
          <Picker.Item label="AUD" value="AUD" />
        </Picker>

        <Text style={styles.label}>To:</Text>
        <Picker
          selectedValue={toCurrency}
          style={styles.picker}
          onValueChange={(value) => setToCurrency(value)}
        >
          <Picker.Item label="USD" value="USD" />
          <Picker.Item label="EUR" value="EUR" />
          <Picker.Item label="GBP" value="GBP" />
          <Picker.Item label="JPY" value="JPY" />
          <Picker.Item label="AUD" value="AUD" />
        </Picker>
      </View>

      <Button title="Convert" onPress={convertCurrency} />

      {convertedAmount && (
        <Text style={styles.resultText}>
          {amount} {fromCurrency} = {convertedAmount} {toCurrency}
        </Text>
      )}

      <TouchableOpacity style={styles.saveButton} onPress={saveFavorite}>
        <Text style={styles.saveButtonText}>Save Favorite</Text>
      </TouchableOpacity>

      <Text style={styles.favoritesTitle}>Favorite Pairs:</Text>
      <FlatList
        data={favorites}
        keyExtractor={(item) => item}
        renderItem={({ item }) => (
          <View style={styles.favoriteItem}>
            <Text>{item}</Text>
            <TouchableOpacity
              style={styles.removeButton}
              onPress={() => removeFavorite(item)}
            >
              <Text style={styles.removeButtonText}>Remove</Text>
            </TouchableOpacity>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  input: {
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    marginBottom: 20,
    width: '100%',
    paddingHorizontal: 10,
    borderRadius: 5,
  },
  pickerContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    marginBottom: 20,
    width: '100%',
  },
  label: {
    fontSize: 16,
  },
  picker: {
    height: 50,
    flex: 1,
  },
  resultText: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 20,
    marginBottom: 20,
  },
  saveButton: {
    backgroundColor: '#4CAF50',
    padding: 10,
    borderRadius: 5,
    marginBottom: 20,
  },
  saveButtonText: {
    color: 'white',
    fontWeight: 'bold',
    textAlign: 'center',
  },
  favoritesTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  favoriteItem: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: 10,
    borderBottomColor: '#ccc',
    borderBottomWidth: 1,
    width: '100%',
  },
  removeButton: {
    backgroundColor: '#f44336',
    padding: 5,
    borderRadius: 5,
  },
  removeButtonText: {
    color: 'white',
  },
});
