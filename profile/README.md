<div align="center">
  <h1>KneeKarePro Release Candidate</h1>
  <h2><i>ACL Recovery Brace</i></h2>
</div>
KneeCarePro is a brace for ACL recovery patients. The patient's brace will be able to upload their rotary data for a clinician to review.

<!-- Table of Contents -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#overview">Overview</a>
    </li>
    <li>
      <a href="#hardware">Hardware</a>
      <ul>
        <li><a href="#hardware-repositories">Hardware Repositories</a></li>
      </ul>
    </li>
    <li>
      <a href="#software">Software</a>
      <ul>
        <li><a href="#data-streamer">Data Streamer</a></li>
        <li><a href="#backend">Backend</a></li>
        <li><a href="#web-application">Web Application</a></li>
      </ul>
    </li>
    <li>
      <a href="#features">Features</a>
    </li>
    <li>
      <a href="#intended-impact">Intended Impact</a>
    </li>
    <li>
      <a href="#repositories">Repositories</a>
    </li>
    <li>
      <a href="#instructions">Instructions</a>
    </li>
    <li>
      <a href="#milestone">Milestone Work</a>
    </li>
    <li>
      <a href="#time">Time Worked</a>
    </li>
    <li>
      <a href="#bugs">Known Bugs and Development Horizon</a>
    </li>
  </ol>
</details>

<div align="center" id="overview">
  <h2>Overview</h2>
</div>
The KneeKarePro is a brace for ACL recovery patients. The brace combines hardware and software to collect a patient's rotary data. The data is then uploaded to a web app for a clinician to review.

<div align="center" id="hardware">
  <h3>Hardware</h3>
</div>

The hardware consists of a ESP32 microcontroller, a LiPo battery, and a potentimeter to measure the rotary data. The firmware on the microcontroller collects, interpolates, and sends notifications to a connected device over Bluetooth Low Energy (BLE). The firmware frameworks utilized are the Arduino framework and the [PlatformIO](https://platformio.org) meta-framework. We decided to use the ESP32 microcontroller because of its built-in BLE capabilities and its low power consumption. The decision to use BLE was made to ensure that the battery life of the device would be long enough to last a full day of use. However, there exists a discussion in shifting the purpose of the device to record and store the data locally on the device. This would allow the device to be used without the need for a connected device. In the event of a user establishing connection with the device the Bluetooth Low Energy communications protocol would no longer be sufficient. In order to improve the bit rate, our connecting device and peripheral device would need to use the Bluetooth Classic protocol.

Currently, we do not have any existing issues or bugs within the hardware or firmware. The hardware and firmware are both in the production stage, and can be easily flashed to a device. Testing firmware and hardware capabilities exist within python scripts which connect and poll data at differing rates. The testing suite includes cases of long wait times in attempt to create timeouts. However, the BLE protocol is resistant to timeout issues due to the nature of request and notifications.

#### Hardware Repositories

<div align="center" id="software">
  <h3>Software</h3>
</div>

<div align="center" id="data-streamer">
  <h4> Data Streamer </h4>
</div>

The **Data Streamer** is the cornerstone of our application and allows the user to establish a connection to the local hardware device and stream data points. The **Data Streamer** simultaneously attempt to reach a user's a backend endpoint, create a user if a user does not exist, and store user data on a persistent database using a user specific table. The currently implementation of the data streamer is a python applications using the `bleak` library to connect to the device and the `requests` library to send data to the backend. The **Data Streamer** is currently in the development stage and due to be migrated to another platform. As mentioned previously, there are existings discussion to reorganize the frequency of communications with the device. If the pivot occurs, the data streamer application will be migrated into a desktop application through the use of [Electron](https://www.electronjs.org/). However, a large number of either existing components within the KneeKarePro project make use of the flexibility and speed of development Python offers. The decision to migrate to Electron would be made in the event of a need for a more robust application however, would likely incorporate Python.

<div align="center" id="backend">
  <h4> Backend </h4>
</div>

The backend handles all user data traffic directing the request and analytics of user data. The backend is a RESTful API built using the [FastAPI](https://fastapi.tiangolo.com/) framework. The backend is currently in the development stage and is on going a migration from the [Flask](https://flask.palletsprojects.com/en/stable/) framework. Currently, our backend endpoints consist of basic CRUD operations for users data in addition to more complex analytical requests handled by `pandas` and `numpy`.

For example, there is a backend endpoint of `/data/stats/{username}` which returns the mean, median, and standard deviation of both angular and rotational speed data.

```python
@app.route('/data/stats/<username>', methods=['GET'])
def get_data_stats(username):
    """
    Retrieve and return statistical data for a given user's device data.
    This function fetches the device data associated with the specified username
    from the database, computes statistical metrics (mean, standard deviation, 
    minimum, and maximum) for the 'angle' and 'rotation' fields, and returns 
    these statistics in a JSON response.
        username (str): The username of the user whose data statistics are to be fetched.
    Returns:
        Response: A JSON response containing the username and the computed statistics 
                  for 'angle' and 'rotation' fields if the user is found, otherwise 
                  an error message with a 404 status code.
    """
    user = User.query.filter_by(username=username).first()
    if not user:
        return jsonify({'error': 'User not found'}), 404

    data = pd.DataFrame([{
        'angle': d.angle,
        'rotation': d.rotation,
        'timestamp': d.timestamp
    } for d in user.device_data])

    stats = {
        'angle': {
            'mean': data['angle'].mean(),
            'std': data['angle'].std(),
            'min': data['angle'].min(),
            'max': data['angle'].max()
        },
        'rotation': {
            'mean': data['rotation'].mean(),
            'std': data['rotation'].std(),
            'min': data['rotation'].min(),
            'max': data['rotation'].max()
        }
    }

    return jsonify({'username': username, 'stats': stats}), 200
```

 Future endpoints will entail more complex analytics such as the rate of change of the data and the rate of change of the rate of change of the data. The numerical manipulations for these endpoints will be handled by `numpy` and `pandas` libraries. Additionally, data persistance is handle through databasing using the `SQLAlchemy` ORM and `SQLite` database. The decision to use `SQLite` was made due to the ease of use and the ability to quickly prototype the backend. However, the decision to use `SQLite` was made with the understanding that the database would be migrated to a more robust database such as `PostgreSQL` or `MySQL` in the future. `SQLite` is not a production grade database and is not intended to be used in a production environment. The decision to migrate to a more robust database was made to ensure that the data would be secure and that the database would be able to handle the large amount of data that would be generated by the device. However, due to the client-server nature of more robust applications, the decision to migrate to a more robust database would be made in the event of a need for a more robust application.

 <div align="center" id="installation-and-usage">
  <h5> Installation and Usage </h5>
</div>

To install the backend, clone the repository and navigate to the `KneeKareBackend` directory. The backend and other Python packages are managed using [Poetry](https://python-poetry.org/) and can be installed using the following command:

```bash
poetry install
```

To run the backend, there is a script describe in the `pyproject.toml` file. The backend can be run using the following command:

```bash
poetry run start
```

To work with the development environment, the backend can be run using the following command:

```bash
poetry shell
```


<div align="center" id="web-application">
  <h4> Web Application
</div>

The web application is the front facing component of the KneeKarePro project. The web application is built using the [React](https://reactjs.org/) framework and is currently in the production stage. The web application is a single page application that allows users to view their data and analytics. The web application utilizese various data visualization libraries such as `chart.js`.

<div align="center" id="features">
  <h2> Features </h2>
</div>

The KneeKarePro system integrates the following key features:
- [x] **Real-time Data Streaming**: The KneeKarePro system is capable of streaming real-time data from the hardware device to the user's connected device.
- [x] **Data Analytics**: The KneeKarePro system is capable of performing data analytics on the user's data, such as computing statistical metrics and generating visualizations.
- [x] **User Interface**: The KneeKarePro system provides a user-friendly interface for clinicians, patients, and other stakeholders to interact with the system.
- [x] **Data Persistence**: The KneeKarePro system is capable of persisting user data to a database for future reference and analysis.
- [x] **Adaptable Hardware**: The KneeKarePro system is designed to be adaptable to different hardware configurations and form factors.
- [x] **Low-Power Consumption**: The KneeKarePro system is designed to be powered by a rechargeable battery for ease of use and portability.

<div align="center">
  <h2> Intended Impact </h2>
</div>

KneeKarePro addresses the need for accurate, continuous data in ACL rehabilitation, allowing clinicians to make data-driven decisions and helping patients stay engaged in their recovery. By providing immediate feedback and a comprehensive tracking solution, KneeKarePro aims to improve rehabilitation outcomes, shorten recovery times, and reduce re-injury risks. The device’s versatility makes it accessible for a wide range of rehabilitation settings and adaptable to various brace types, potentially expanding its use across different physical therapy applications.

<div align="center" id="repositories">
  <h2> Repositories </h2>
</div>

- [Firmware Repository](https://github.com/KneeKarePro/KneeKareProFirmware)
- [Hardware + STL Repository]()
- [Data Streamer Repository]()
- [Backend Repository]()
- [Web Application Repository]()

<div align="center" id="instructions">
  <h2> Instructions </h2>
</div>


<div align="center" id="milestone">
  <h2> Milestone Work </h2>
</div>

<div align="center" id="time">
  <h2> Time Worked </h2>
</div>

<div align="center" id="bugs">
  <h2> Known Bugs and Development Horizon </h2>
</div>