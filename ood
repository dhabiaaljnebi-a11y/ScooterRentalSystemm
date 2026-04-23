from datetime import datetime


class User:
    "Base class for all users"

    def __init__(self, user_id="", name="", email=""):
        self._user_id = user_id
        self._name = name
        self._email = email

    def get_user_id(self):
        return self._user_id

    def get_name(self):
        return self._name

    def get_email(self):
        return self._email

    def view_scooter_availability(self, station):
        return station.view_available_scooters()

    def view_station_location(self, station):
        return f"{station.get_name()} - {station.get_location()}"

    def __str__(self):
        return f"User ID: {self._user_id}, Name: {self._name}, Email: {self._email}"


class GuestUser(User):
    "Guest user can register"

    def register(self):
        return RegisteredUser(self.get_user_id(), self.get_name(), self.get_email())


class RegisteredUser(User):
    "Registered user can reserve, rent, return, and report issues"

    def __init__(self, user_id="", name="", email=""):
        super().__init__(user_id, name, email)
        self._rental_history = []
        self._reserved_scooter = None

    def reserve_scooter(self, scooter):
        if scooter.get_status() == "available":
            scooter.set_status("reserved")
            self._reserved_scooter = scooter
            return f"Scooter {scooter.get_scooter_id()} reserved successfully."
        return f"Scooter {scooter.get_scooter_id()} cannot be reserved."

    def unlock_scooter(self, scooter):
        if scooter.get_status() == "reserved" and self._reserved_scooter == scooter:
            return f"Scooter {scooter.get_scooter_id()} unlocked successfully."
        elif scooter.get_status() == "available":
            return f"Scooter {scooter.get_scooter_id()} unlocked successfully."
        return f"Scooter {scooter.get_scooter_id()} cannot be unlocked."

    def rent_scooter(self, scooter):
        if scooter.get_status() == "available" or (
            scooter.get_status() == "reserved" and self._reserved_scooter == scooter
        ):
            scooter.set_status("in use")
            rental = Rental(self, scooter)
            rental.start_rental()
            self._rental_history.append(rental)

            if self._reserved_scooter == scooter:
                self._reserved_scooter = None

            return rental
        return None

    def return_scooter(self, rental, station):
        if rental in self._rental_history and rental.get_scooter().get_status() == "in use":
            rental.end_rental()
            rental.get_scooter().set_status("available")
            rental.get_scooter().set_location(station.get_name())
            return station.add_scooter(rental.get_scooter())
        return "Return failed."

    def view_rental_history(self):
        if len(self._rental_history) == 0:
            return "No rental history."
        output = ""
        for rental in self._rental_history:
            output += str(rental) + "\n"
        return output

    def report_issue(self, scooter, issue_description):
        scooter.set_status("maintenance")
        record = MaintenanceRecord(scooter, issue_description)
        scooter.add_maintenance_record(record)
        return f"Issue reported for scooter {scooter.get_scooter_id()}: {issue_description}"


class Scooter:
    "Scooter class"

    def __init__(self, scooter_id="", battery_level=0, scooter_type="standard", location=""):
        self._scooter_id = scooter_id
        self._battery_level = battery_level
        self._scooter_type = scooter_type
        self._status = "available"
        self._location = location
        self._maintenance_records = []

    def get_scooter_id(self):
        return self._scooter_id

    def get_battery_level(self):
        return self._battery_level

    def get_scooter_type(self):
        return self._scooter_type

    def get_status(self):
        return self._status

    def get_location(self):
        return self._location

    def get_maintenance_records(self):
        return self._maintenance_records

    def set_status(self, status):
        self._status = status

    def set_location(self, location):
        self._location = location

    def add_maintenance_record(self, record):
        self._maintenance_records.append(record)

    def __str__(self):
        return (
            f"Scooter ID: {self._scooter_id}, Battery: {self._battery_level}%, "
            f"Type: {self._scooter_type}, Status: {self._status}, Location: {self._location}"
        )


class Station:
    "Station class"

    def __init__(self, station_id="", name="", location="", capacity=0):
        self._station_id = station_id
        self._name = name
        self._location = location
        self._capacity = capacity
        self._scooters = []

    def get_station_id(self):
        return self._station_id

    def get_name(self):
        return self._name

    def get_location(self):
        return self._location

    def get_capacity(self):
        return self._capacity

    def get_scooters(self):
        return self._scooters

    def add_scooter(self, scooter):
        if scooter in self._scooters:
            return f"Scooter {scooter.get_scooter_id()} is already in {self._name}."

        if len(self._scooters) < self._capacity:
            self._scooters.append(scooter)
            scooter.set_location(self._name)
            return f"Scooter {scooter.get_scooter_id()} added to {self._name}."
        return f"Station {self._name} is full."

    def remove_scooter(self, scooter):
        if scooter in self._scooters:
            self._scooters.remove(scooter)
            return f"Scooter {scooter.get_scooter_id()} removed from {self._name}."
        return f"Scooter {scooter.get_scooter_id()} not found in {self._name}."

    def view_available_scooters(self):
        available = []
        for scooter in self._scooters:
            if scooter.get_status() == "available":
                available.append(str(scooter))

        if len(available) == 0:
            return "No available scooters."
        return "\n".join(available)

    def __str__(self):
        return (
            f"Station ID: {self._station_id}, Name: {self._name}, "
            f"Location: {self._location}, Capacity: {self._capacity}"
        )


class Rental:
    "Rental class"

    def __init__(self, user, scooter):
        self._user = user
        self._scooter = scooter
        self._start_time = None
        self._end_time = None
        self._total_cost = 0.0

    def get_user(self):
        return self._user

    def get_scooter(self):
        return self._scooter

    def get_start_time(self):
        return self._start_time

    def get_end_time(self):
        return self._end_time

    def get_total_cost(self):
        return self._total_cost

    def start_rental(self):
        self._start_time = datetime.now()

    def end_rental(self):
        self._end_time = datetime.now()
        self._total_cost = self.calculate_cost()

    def calculate_cost(self):
        if self._start_time is None or self._end_time is None:
            return 0.0

        duration_minutes = (self._end_time - self._start_time).total_seconds() / 60

        if self._scooter.get_scooter_type().lower() == "premium":
            rate_per_minute = 1.5
        else:
            rate_per_minute = 1.0

        return round(duration_minutes * rate_per_minute, 2)

    def __str__(self):
        return (
            f"Rental - User: {self._user.get_name()}, Scooter: {self._scooter.get_scooter_id()}, "
            f"Start: {self._start_time}, End: {self._end_time}, Cost: AED {self._total_cost}"
        )


class MaintenanceRecord:
    "Maintenance record for a scooter"

    def __init__(self, scooter, issue_description):
        self._scooter = scooter
        self._issue_description = issue_description
        self._date_reported = datetime.now()
        self._status = "open"

    def get_scooter(self):
        return self._scooter

    def get_issue_description(self):
        return self._issue_description

    def get_date_reported(self):
        return self._date_reported

    def get_status(self):
        return self._status

    def mark_resolved(self):
        self._status = "resolved"
        self._scooter.set_status("available")

    def __str__(self):
        return (
            f"Maintenance Record - Scooter: {self._scooter.get_scooter_id()}, "
            f"Issue: {self._issue_description}, Date: {self._date_reported}, Status: {self._status}"
        )


class ScooterRentalSystem:
    "System class to manage users, stations, scooters, and rentals"

    def __init__(self):
        self._users = []
        self._stations = []
        self._scooters = []
        self._rentals = []

    def add_user(self, user):
        self._users.append(user)

    def add_station(self, station):
        self._stations.append(station)

    def add_scooter(self, scooter, station):
        self._scooters.append(scooter)
        return station.add_scooter(scooter)

    def create_rental(self, user, scooter, station):
        if scooter.get_status() == "maintenance":
            return None

        if scooter in station.get_scooters():
            station.remove_scooter(scooter)

        rental = user.rent_scooter(scooter)

        if rental is not None:
            self._rentals.append(rental)
            return rental
        return None


# Test the system
if __name__ == "__main__":
    system = ScooterRentalSystem()

    station1 = Station("ST01", "Main Station", "Abu Dhabi", 5)
    station2 = Station("ST02", "City Station", "Al Ain", 5)

    system.add_station(station1)
    system.add_station(station2)

    scooter1 = Scooter("SC01", 90, "standard", "Main Station")
    scooter2 = Scooter("SC02", 80, "premium", "Main Station")
    scooter3 = Scooter("SC03", 70, "standard", "City Station")

    print(system.add_scooter(scooter1, station1))
    print(system.add_scooter(scooter2, station1))
    print(system.add_scooter(scooter3, station2))

    guest1 = GuestUser("U01", "Sara", "sara@email.com")
    user1 = guest1.register()
    system.add_user(user1)

    print("\nAvailable scooters at Main Station:")
    print(user1.view_scooter_availability(station1))

    print("\nReserve scooter:")
    print(user1.reserve_scooter(scooter1))

    print("\nUnlock scooter:")
    print(user1.unlock_scooter(scooter1))

    print("\nRent scooter:")
    rental1 = system.create_rental(user1, scooter1, station1)
    if rental1 is not None:
        print("Rental started successfully.")
    else:
        print("Rental failed.")

    print("\nReturn scooter:")
    if rental1 is not None:
        print(user1.return_scooter(rental1, station2))

    print("\nRental history:")
    print(user1.view_rental_history())

    print("\nReport issue:")
    print(user1.report_issue(scooter2, "Brake problem"))

    print("\nMaintenance records:")
    for record in scooter2.get_maintenance_records():
        print(record)
