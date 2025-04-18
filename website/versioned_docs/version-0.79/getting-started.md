npx create-expo-app GrooveChatMobile
cd GrooveChatMobile

npm install @react-navigation/native @react-navigation/bottom-tabs react-native-screens react-native-safe-area-context react-native-gesture-handler react-native-reanimated react-native-vector-icons emoji-mart-native

# Expo-specific dependencies
npx expo install expo-status-bar
GrooveChatMobile/
├── App.js
├── assets/
├── components/
│   ├── ChatBubble.js
│   └── EventCard.js
├── screens/
│   ├── ExploreScreen.js
│   ├── EventsScreen.js
│   ├── ChatScreen.js
│   └── ProfileScreen.js
└── utils/
    └── websocket.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import ExploreScreen from './screens/ExploreScreen';
import EventsScreen from './screens/EventsScreen';
import ChatScreen from './screens/ChatScreen';
import ProfileScreen from './screens/ProfileScreen';

const Tab = createBottomTabNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={{
          headerShown: false,
          tabBarStyle: { backgroundColor: '#111' },
          tabBarActiveTintColor: '#fff',
          tabBarInactiveTintColor: '#888',
        }}
      >
        <Tab.Screen name="Explore" component={ExploreScreen} />
        <Tab.Screen name="Events" component={EventsScreen} />
        <Tab.Screen name="Chat" component={ChatScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TextInput, StyleSheet } from 'react-native';
import EventCard from '../components/EventCard';

const ExploreScreen = () => {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    fetch('https://your-api-url.com/api/events?tab=nearby')
      .then(res => res.json())
      .then(data => setEvents(data.events || []));
  }, []);

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.header}>GrooveChat</Text>
      <TextInput
        style={styles.input}
        placeholder="Search events..."
        placeholderTextColor="#888"
      />
      {events.map((event, i) => (
        <EventCard key={i} event={event} />
      ))}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000', padding: 16 },
  header: { fontSize: 28, fontWeight: 'bold', color: '#fff', marginBottom: 16 },
  input: {
    backgroundColor: '#1A1A1A',
    borderRadius: 8,
    padding: 10,
    marginBottom: 16,
    color: '#fff',
  },
});

export default ExploreScreen;
// utils/websocket.js
import { useEffect, useRef } from 'react';

export default function useChatWebSocket(onMessageReceived) {
  const ws = useRef(null);

  useEffect(() => {
    ws.current = new WebSocket('wss://echo.websocket.events');

    ws.current.onopen = () => {
      console.log('✅ Connected to WebSocket');
    };

    ws.current.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'chat-message') {
        onMessageReceived(data.data);
      }
    };

    ws.current.onerror = (e) => console.error('WebSocket Error', e.message);
    ws.current.onclose = () => console.log('❌ WebSocket Disconnected');

    return () => ws.current.close();
  }, []);

  const sendMessage = (msg) => {
    const payload = JSON.stringify({ type: 'chat-message', data: msg });
    ws.current.send(payload);
  };

  return { sendMessage };
}
// components/EventCard.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const EventCard = ({ event }) => (
  <View style={styles.card}>
    <Text style={styles.title}>{event.title}</Text>
    <Text style={styles.detail}>{event.artist}</Text>
    <Text style={styles.detail}>{event.date}</Text>
    <Text style={styles.detail}>{event.venue}</Text>
  </View>
);

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#1A1A1A',
    borderRadius: 10,
    padding: 16,
    marginBottom: 12,
  },
  title: { fontSize: 18, fontWeight: 'bold', color: '#fff' },
  detail: { fontSize: 14, color: '#aaa' },
});

export default EventCard;
// screens/ChatScreen.js
import React, { useState } from 'react';
import { View, Text, TextInput, Button, FlatList, StyleSheet, TouchableOpacity } from 'react-native';
import useChatWebSocket from '../utils/websocket';
import EmojiPicker from 'emoji-mart-native';

const ChatScreen = () => {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [showEmoji, setShowEmoji] = useState(false);

  const addMessage = (msg) => setMessages((prev) => [...prev, msg]);

  const { sendMessage } = useChatWebSocket(addMessage);

  const handleSend = () => {
    if (!input.trim()) return;
    const msgObj = {
      sender: 'PartySteve',
      message: input,
    };
    sendMessage(msgObj);
    addMessage(msgObj);
    setInput('');
  };

  return (
    <View style={styles.container}>
      <FlatList
        data={messages}
        keyExtractor={(_, index) => index.toString()}
        renderItem={({ item }) => (
          <Text style={styles.message}><Text style={styles.sender}>{item.sender}:</Text> {item.message}</Text>
        )}
      />
      {showEmoji && (
        <EmojiPicker onEmojiSelected={(emoji) => setInput((prev) => prev + emoji.native)} />
      )}
      <View style={styles.inputRow}>
        <TextInput
          value={input}
          onChangeText={setInput}
          placeholder="Type a message..."
          placeholderTextColor="#777"
          style={styles.input}
        />
        <TouchableOpacity onPress={() => setShowEmoji(!showEmoji)}>
          <Text style={styles.emojiToggle}>😀</Text>
        </TouchableOpacity>
        <Button title="Send" onPress={handleSend} />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000', padding: 12 },
  message: { color: '#fff', marginVertical: 4 },
  sender: { color: '#0af' },
  inputRow: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: 12,
    gap: 8,
  },
  input: {
    flex: 1,
    backgroundColor: '#1A1A1A',
    color: '#fff',
    padding: 10,
    borderRadius: 6,
  },
  emojiToggle: { fontSize: 22, paddingHorizontal: 6 },
});

export default ChatScreen;
// screens/EventsScreen.js
import React, { useEffect, useState } from 'react';
import { ScrollView, Text, StyleSheet } from 'react-native';
import EventCard from '../components/EventCard';

const EventsScreen = () => {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    // Mocked Events
    const fakeEvents = [
      { title: "Neon Bass Night", artist: "DJ Pulse", date: "April 21", venue: "Glow Club" },
      { title: "Sunset Vibes", artist: "Luna Live", date: "April 22", venue: "Skypark" },
      { title: "Deep Groove Techno", artist: "Beatsmith", date: "April 23", venue: "SubTunnel" },
    ];
    setEvents(fakeEvents);
  }, []);

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.header}>Your Events</Text>
      {events.map((event, idx) => <EventCard key={idx} event={event} />)}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000', padding: 16 },
  header: { fontSize: 24, fontWeight: 'bold', color: '#fff', marginBottom: 12 },
});

export default EventsScreen;
