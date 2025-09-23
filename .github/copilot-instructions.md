# ioBroker Adapter Development with GitHub Copilot

**Version:** 0.4.0
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on ioBroker adapter development.

## Project Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

### Netatmo-Crawler Adapter Specific Context

This adapter crawls weather information from public Netatmo weather stations. It connects to the Netatmo weather map service to retrieve real-time weather data including temperature, humidity, pressure, wind, and precipitation data from publicly shared weather stations.

**Key Features:**
- Fetches data from multiple Netatmo weather stations using public URLs
- Runs on a configurable schedule (default: every 5 minutes)
- Supports multiple station configurations with flexible naming
- Provides weather data states for home automation systems

**Core Dependencies:**
- `@iobroker/adapter-core`: Standard ioBroker adapter framework
- `request`: HTTP client for API calls to Netatmo services
- `moment`: Date/time handling for weather data timestamps
- `url`: URL parsing for Netatmo station endpoints

**Configuration:**
- `stationUrls`: Comma-separated list of Netatmo weather station URLs
- `stationNameType`: Naming convention for weather stations (counter or custom)

## Adapter-Specific Development Patterns

### Netatmo API Integration
When working with Netatmo weather station APIs:
- Station URLs follow the pattern: `https://weathermap.netatmo.com/...`
- Use regex pattern: `/(https:\/\/weathermap\.netatmo\.com\/{1,}[-a-zA-Z0-9()@:%_+.~#?&//=]*)/g`
- Extract station IDs from URLs using `getStationId()` method
- Handle authentication tokens via `getAuthorizationToken()` method

### Weather Data Processing
- Weather measurements include: temperature, humidity, pressure, wind_strength, wind_angle, gust_strength, gust_angle, rain, rain_lastHour
- All numeric values should be rounded to 2 decimal places
- Timestamps are converted using moment.js for proper formatting
- Save states with acknowledgment flag set to true for read-only data

### Error Handling Patterns
- Wrap station data fetching in try-catch blocks
- Log warnings for individual station failures without stopping entire process
- Continue processing remaining stations if one fails
- Use appropriate log levels: debug for verbose info, warn for recoverable errors, error for critical failures

### State Management
- Create states with proper units and roles (e.g., temperature with °C unit and "value.temperature" role)
- Use `createOwnState()` method for custom state creation
- Save measurements using `saveMeasure()` and timestamps using `saveTimestamp()`
- Mark all weather data states as read-only

## Testing

### Unit Testing
- Use Jest as the primary testing framework for ioBroker adapters
- Test files should be named `*.test.js`
- Integration tests go in the `test/` directory
- Focus test coverage on:
  - Station URL parsing and validation
  - Weather data processing and state creation
  - Error handling for network failures
  - Authentication token management

### Integration Testing
- Use `@iobroker/testing` framework for adapter integration tests
- Test actual adapter startup and configuration
- Validate state creation and data flow
- Test adapter cleanup and shutdown procedures

## Dependencies

### Core Dependencies
```javascript
const utils = require('@iobroker/adapter-core');
const url = require('url');
const moment = require('moment');
const request = require('request');
```

### Development Dependencies
- `@iobroker/testing`: Integration testing framework
- `@iobroker/eslint-config`: ESLint configuration for ioBroker
- `mocha`: Test runner
- `chai`: Assertion library

## Common Patterns

### Adapter Class Structure
```javascript
class NetatmoCrawler extends utils.Adapter {
    constructor(options) {
        super({ ...options, name: 'netatmo-crawler' });
        this.on('ready', this.onReady.bind(this));
        this.on('unload', this.onUnload.bind(this));
    }

    async onReady() {
        // Initialize adapter
        // Parse station URLs
        // Fetch and process weather data
        // Terminate with success
    }

    onUnload(callback) {
        // Clean up resources
        callback();
    }
}
```

### State Creation Pattern
```javascript
await this.createOwnState(stateName, unit, 'number', 'value.temperature');
await this.saveMeasure(id, measureName, measureValue);
```

### HTTP Request Pattern
```javascript
const options = {
    url: stationUrl,
    headers: { 'Authorization': `Bearer ${token}` },
    timeout: 10000
};

request(options, (error, response, body) => {
    if (error) {
        this.log.warn(`Request failed: ${error.message}`);
        return;
    }
    // Process response
});
```

## Configuration Management

### io-package.json Configuration
- Mode: "schedule" for timed execution
- Schedule: "*/5 * * * *" (every 5 minutes)
- Type: "weather" for weather-related adapters
- ConnectionType: "cloud" for internet-based services
- DataSource: "poll" for scheduled data retrieval

### Admin Configuration
- Uses materialize UI framework
- Configuration stored in `native` section
- Validate URLs match Netatmo weathermap pattern
- Support multiple station URLs separated by newlines or commas

## Logging Best Practices

### Log Levels
- `debug`: Detailed information for development (station URLs, processing steps)
- `info`: General information about adapter operation
- `warn`: Non-critical issues (individual station failures)
- `error`: Critical errors that affect adapter functionality

### Example Logging
```javascript
this.log.debug(`Working with stationUrl: ${stationUrl}`);
this.log.warn(`Could not work with station ${counter + 1} - Message: ${e}`);
this.log.error(`Authentication failed: ${error.message}`);
```

## Security Considerations

### API Security
- Never log authentication tokens in plaintext
- Handle rate limiting from Netatmo API
- Implement exponential backoff for failed requests
- Validate all URL inputs to prevent injection attacks

### Data Privacy
- Only access publicly shared weather stations
- Respect Netatmo's terms of service
- Don't cache sensitive authentication data longer than necessary

## Performance Optimization

### Request Efficiency
- Process stations sequentially to avoid overwhelming the API
- Implement request timeouts (10 seconds recommended)
- Use appropriate User-Agent headers for identification
- Cache authentication tokens for the session duration

### Memory Management
- Clean up HTTP requests properly
- Avoid memory leaks in long-running scheduled operations
- Terminate adapter gracefully after data collection

## Deployment and Distribution

### Package Structure
- Main adapter code in `main.js`
- Admin configuration in `admin/` directory
- Documentation in `README.md`
- Changelog in `CHANGELOG_OLD.md`

### Version Management
- Follow semantic versioning (semver)
- Update version in both `package.json` and `io-package.json`
- Document changes in news section of io-package.json
- Support multiple languages in news entries

## Code Quality

### ESLint Configuration
- Use `@iobroker/eslint-config` for consistent code style
- Address JSDoc warnings for better documentation
- Follow TypeScript-style parameter documentation
- Maintain consistent indentation and formatting

### TypeScript Support
- Include TypeScript definitions for better IDE support
- Use `lib/adapter-config.d.ts` for configuration types
- Maintain compatibility with both JavaScript and TypeScript

This template provides comprehensive guidance for developing and maintaining the Netatmo-Crawler ioBroker adapter with GitHub Copilot assistance.