# Weather-report
#A weather report is a software application that provides weather information for a specific location and time
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.geom.RoundRectangle2D;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URI;
import java.net.URL;

public class WeatherWiz extends JFrame {

    private JTextField cityField;
    private JComboBox<String> languageComboBox;
    private JLabel weatherLabel, tempLabel, windSpeedLabel, humidityLabel, uvLabel, iconLabel;
    private JCheckBox notificationCheckBox;

    public WeatherWiz() {
        setTitle("Weather App");
        setSize(600, 700);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);

        // Setting up the layout
        setLayout(new BorderLayout());

        // Background panel
        JPanel backgroundPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                ImageIcon backgroundIcon = new ImageIcon("//path of backgroundimage.jpg");
                Image backgroundImage = backgroundIcon.getImage();
                g.drawImage(backgroundImage, 0, 0, getWidth(), getHeight(), this);
            }
        };
        backgroundPanel.setLayout(new BorderLayout());

        // Top panel for entering the city
        JPanel topPanel = new JPanel();
        topPanel.setOpaque(false);
        topPanel.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(10, 10, 10, 10);

        JLabel cityLabel = new JLabel("Enter City: ");
        cityLabel.setFont(new Font("Arial", Font.BOLD, 18));
        cityLabel.setForeground(Color.WHITE);
        gbc.gridx = 0;
        gbc.gridy = 0;
        topPanel.add(cityLabel, gbc);

        cityField = new RoundedTextField(20); // Customized rounded text field
        cityField.setFont(new Font("Arial", Font.PLAIN, 18));
        gbc.gridx = 1;
        gbc.gridy = 0;
        topPanel.add(cityField, gbc);

        JLabel langLabel = new JLabel("Language: ");
        langLabel.setFont(new Font("Arial", Font.BOLD, 18));
        langLabel.setForeground(Color.WHITE);
        gbc.gridx = 0;
        gbc.gridy = 1;
        topPanel.add(langLabel, gbc);

        languageComboBox = new JComboBox<>(new String[]{"en", "es", "fr", "de"}); // Add more languages as needed
        languageComboBox.setFont(new Font("Arial", Font.PLAIN, 18));
        languageComboBox.setBackground(Color.WHITE);
        gbc.gridx = 1;
        gbc.gridy = 1;
        topPanel.add(languageComboBox, gbc);

        JButton searchButton = new JButton("Search");
        searchButton.setFont(new Font("Arial", Font.BOLD, 18));
        searchButton.setBackground(Color.BLUE);
        searchButton.setForeground(Color.WHITE);
        gbc.gridx = 0;
        gbc.gridy = 2;
        gbc.gridwidth = 2;
        searchButton.addActionListener(new SearchButtonListener());
        topPanel.add(searchButton, gbc);

        notificationCheckBox = new JCheckBox("Enable Notifications");
        notificationCheckBox.setFont(new Font("Arial", Font.BOLD, 14));
        notificationCheckBox.setForeground(Color.WHITE);
        notificationCheckBox.setOpaque(false);
        gbc.gridx = 0;
        gbc.gridy = 3;
        gbc.gridwidth = 2;
        topPanel.add(notificationCheckBox, gbc);

        // Center panel for displaying weather info
        JPanel centerPanel = new JPanel();
        centerPanel.setOpaque(false);
        centerPanel.setLayout(new GridBagLayout());

        gbc.gridx = 0;
        gbc.gridy = 0;
        gbc.gridwidth = 2;
        iconLabel = new JLabel();
        centerPanel.add(iconLabel, gbc);

        gbc.gridy = 1;
        weatherLabel = new JLabel("Weather: N/A", SwingConstants.CENTER);
        weatherLabel.setFont(new Font("Arial", Font.BOLD, 24));
        weatherLabel.setForeground(Color.WHITE);
        centerPanel.add(weatherLabel, gbc);

        gbc.gridy = 2;
        tempLabel = new JLabel("Temperature: N/A", SwingConstants.CENTER);
        tempLabel.setFont(new Font("Arial", Font.BOLD, 24));
        tempLabel.setForeground(Color.WHITE);
        centerPanel.add(tempLabel, gbc);

        gbc.gridy = 3;
        windSpeedLabel = new JLabel("Wind Speed: N/A", SwingConstants.CENTER);
        windSpeedLabel.setFont(new Font("Arial", Font.BOLD, 24));
        windSpeedLabel.setForeground(Color.WHITE);
        centerPanel.add(windSpeedLabel, gbc);

        gbc.gridy = 4;
        humidityLabel = new JLabel("Humidity: N/A", SwingConstants.CENTER);
        humidityLabel.setFont(new Font("Arial", Font.BOLD, 24));
        humidityLabel.setForeground(Color.WHITE);
        centerPanel.add(humidityLabel, gbc);

        gbc.gridy = 5;
        uvLabel = new JLabel("UV Index: N/A", SwingConstants.CENTER);
        uvLabel.setFont(new Font("Arial", Font.BOLD, 24));
        uvLabel.setForeground(Color.WHITE);
        centerPanel.add(uvLabel, gbc);

        backgroundPanel.add(topPanel, BorderLayout.NORTH);
        backgroundPanel.add(centerPanel, BorderLayout.CENTER);
        add(backgroundPanel);
    }

    private class SearchButtonListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            String city = cityField.getText().trim();
            String language = languageComboBox.getSelectedItem().toString();
            if (!city.isEmpty()) {
                fetchWeatherData(city, language);
                if (notificationCheckBox.isSelected()) {
                    JOptionPane.showMessageDialog(null, "Weather data fetched successfully!");
                }
            } else {
                JOptionPane.showMessageDialog(null, "Please enter a city name.");
            }
        }
    }

    private void fetchWeatherData(String city, String language) {
        String apiKey = ""; // Replace with your OpenWeatherMap API key
        String apiUrl = "http://api.openweathermap.org/data/2.5/weather?q=" + city + "&appid=" + apiKey + "&units=metric&lang=" + language;

        try {
            URI uri = new URI(apiUrl);
            URL url = uri.toURL();
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");

            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String line;
            StringBuilder response = new StringBuilder();
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            reader.close();

            String responseString = response.toString();
            String weatherDescription = extractData(responseString, "\"description\":\"", "\"");
            String temperature = extractData(responseString, "\"temp\":", ",");
            String windSpeed = extractData(responseString, "\"speed\":", ",");
            String humidity = extractData(responseString, "\"humidity\":", ",");
            String uvIndex = fetchUVIndex(city, apiKey);
            String iconCode = extractData(responseString, "\"icon\":\"", "\"");

            weatherLabel.setText("Weather: " + weatherDescription);
            tempLabel.setText("Temperature: " + temperature + "°C");
            windSpeedLabel.setText("Wind Speed: " + windSpeed + " m/s");
            humidityLabel.setText("Humidity: " + humidity + "%");
            uvLabel.setText("UV Index: " + uvIndex);

            String iconUrl = "http://openweathermap.org/img/wn/" + iconCode + "@2x.png";
            @SuppressWarnings("deprecation")
            ImageIcon icon = new ImageIcon(new URL(iconUrl));
            iconLabel.setIcon(icon);

        } catch (Exception ex) {
            JOptionPane.showMessageDialog(null, "Failed to get weather data.");
            ex.printStackTrace();
        }
    }

    private String fetchUVIndex(String city, String apiKey) {
        String lat = "28.6139"; // Latitude for New Delhi
        String lon = "77.2090"; // Longitude for New Delhi
        String apiUrl = "http://api.openweathermap.org/data/2.5/uvi?appid=" + apiKey + "&lat=" + lat + "&lon=" + lon;

        try {
            URI uri = new URI(apiUrl);
            URL url = uri.toURL();
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");

            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String line;
            StringBuilder response = new StringBuilder();
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            reader.close();

            String responseString = response.toString();
            return extractData(responseString, "\"value\":", "}");

        } catch (Exception ex) {
            ex.printStackTrace();
            return "N/A";
        }
    }

    private String extractData(String json, String startDelimiter, String endDelimiter) {
        int startIndex = json.indexOf(startDelimiter) + startDelimiter.length();
        int endIndex = json.indexOf(endDelimiter, startIndex);
        return json.substring(startIndex, endIndex);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            WeatherWiz app = new WeatherWiz();
            app.setVisible(true);
        });
    }
}

class RoundedTextField extends JTextField {
    private Shape shape;

    public RoundedTextField(int size) {
        super(size);
        setOpaque(false);
    }

    protected void paintComponent(Graphics g) {
        g.setColor(getBackground());
        g.fillRoundRect(0, 0, getWidth()-1, getHeight()-1, 15, 15);
        super.paintComponent(g);
    }

    protected void paintBorder(Graphics g) {
        g.setColor(getForeground());
        g.drawRoundRect(0, 0, getWidth()-1, getHeight()-1, 15, 15);
    }

    public boolean contains(int x, int y) {
        if (shape == null || !shape.getBounds().equals(getBounds())) {
            shape = new RoundRectangle2D.Float(0, 0, getWidth(), getHeight(), 15, 15);
        }
        return shape.contains(x, y);
    }
}
