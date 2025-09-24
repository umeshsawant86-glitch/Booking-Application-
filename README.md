import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

// Function to generate time slots
function generateSlots(startHour, endHour, interval = 15) {
  const slots = [];
  for (let hour = startHour; hour < endHour; hour++) {
    for (let min = 0; min < 60; min += interval) {
      if (hour === endHour - 1 && min >= (endHour * 60) % 60) break;
      const h = hour % 12 === 0 ? 12 : hour % 12;
      const ampm = hour < 12 ? "AM" : "PM";
      slots.push(`${h}:${min === 0 ? "00" : min} ${ampm}`);
    }
  }
  return slots;
}

// Generate slots for Quarry Road Clinic
function getQuarrySlots(date) {
  const day = new Date(date).getDay();
  if (!date) return [];

  const afternoon = generateSlots(12, 14).concat(["2:15 PM", "2:30 PM"]);
  let evening = generateSlots(19, 22);

  if (day === 0) {
    // Sunday, evening closed
    return afternoon;
  }
  return [...afternoon, ...evening];
}

// Generate slots for Asian Paint Clinic
function getAsianPaintSlots(date) {
  const day = new Date(date).getDay();
  if (!date) return [];

  const evening = generateSlots(16, 19).concat(["7:15 PM", "7:30 PM"]);

  if (day === 0) {
    // Sunday special timing
    return generateSlots(11, 13);
  }
  return evening;
}

export default function TulsiClinicApp() {
  const [selectedClinic, setSelectedClinic] = useState("");
  const [selectedDate, setSelectedDate] = useState("");
  const [patientName, setPatientName] = useState("");
  const [patientNumber, setPatientNumber] = useState("");
  const [appointmentType, setAppointmentType] = useState("");
  const [bookings, setBookings] = useState({});

  const getSlots = () => {
    if (!selectedDate || !selectedClinic) return [];
    if (selectedClinic === "quarry") return getQuarrySlots(selectedDate);
    if (selectedClinic === "asian") return getAsianPaintSlots(selectedDate);
    return [];
  };

  const handleBooking = (slot) => {
    if (!selectedDate || !patientName.trim() || !patientNumber.trim() || !appointmentType || !selectedClinic) return;

    const key = `${selectedClinic}_${selectedDate}`;
    const bookedForDate = bookings[key] || [];

    // Prevent double booking by same patient on the same date
    const alreadyBookedByPatient = bookedForDate.some(
      (b) => b.name.toLowerCase() === patientName.toLowerCase()
    );

    if (!bookedForDate.find((b) => b.slot === slot) && !alreadyBookedByPatient) {
      setBookings({
        ...bookings,
        [key]: [
          ...bookedForDate,
          { slot, name: patientName, number: patientNumber, type: appointmentType },
        ],
      });
      setPatientName("");
      setPatientNumber("");
      setAppointmentType("");
    }
  };

  const handleCancel = (key, slot) => {
    const updated = bookings[key].filter((b) => b.slot !== slot);
    setBookings({ ...bookings, [key]: updated });
  };

  const isBooked = (slot) => {
    if (!selectedDate || !selectedClinic) return false;
    const key = `${selectedClinic}_${selectedDate}`;
    return bookings[key]?.some((b) => b.slot === slot);
  };

  const clinicName = (id) => {
    if (id === "quarry") return "Tulsi Kids Clinic, Quarry Road (Near Mangathram Petrol Pump, Bhandup West)";
    if (id === "asian") return "Tulsi Kids Clinic (Asian Paint), Shop No. 34, Mayuresh Shristi, Bhandup West";
    return "";
  };

  return (
    <div className="min-h-screen bg-gray-50 flex flex-col items-center justify-center p-6 space-y-6">
      <Card className="w-full max-w-3xl shadow-lg bg-white border border-gray-200">
        <CardHeader className="bg-gray-100 rounded-t-xl">
          <CardTitle className="text-2xl font-bold text-center text-gray-800">
            Tulsi Kids Clinic - Appointment Booking
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div className="mb-4 flex flex-col space-y-4 items-center">
            <Select value={selectedClinic} onValueChange={setSelectedClinic}>
              <SelectTrigger className="w-2/3 bg-gray-50 border-gray-300">
                <SelectValue placeholder="Select Clinic Location" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="quarry">Quarry Road, Near Mangathram Petrol Pump</SelectItem>
                <SelectItem value="asian">Asian Paint, Mayuresh Shristi</SelectItem>
              </SelectContent>
            </Select>
            <Input
              type="date"
              value={selectedDate}
              onChange={(e) => setSelectedDate(e.target.value)}
              className="w-2/3 border-gray-300"
            />
            <Input
              type="text"
              placeholder="Enter patient number"
              value={patientNumber}
              onChange={(e) => setPatientNumber(e.target.value)}
              className="w-2/3 border-gray-300"
            />
            <Input
              type="text"
              placeholder="Enter patient name"
              value={patientName}
              onChange={(e) => setPatientName(e.target.value)}
              className="w-2/3 border-gray-300"
            />
            <Select value={appointmentType} onValueChange={setAppointmentType}>
              <SelectTrigger className="w-2/3 bg-gray-50 border-gray-300">
                <SelectValue placeholder="Select appointment type" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="first">First Consultation</SelectItem>
                <SelectItem value="followup">Follow-up Consultation</SelectItem>
              </SelectContent>
            </Select>
          </div>

          {!selectedClinic && (
            <p className="text-center text-gray-500 mb-4">
              Please select a clinic to view available slots.
            </p>
          )}

          <div className="grid grid-cols-2 gap-4">
            {getSlots().map((slot, index) => (
              <Button
                key={index}
                onClick={() => handleBooking(slot)}
                disabled={
                  !selectedDate ||
                  !patientName.trim() ||
                  !patientNumber.trim() ||
                  !appointmentType ||
                  !selectedClinic ||
                  isBooked(slot)
                }
                className={`p-4 rounded-2xl shadow text-white transition-colors duration-200 ${
                  isBooked(slot)
                    ? "bg-gray-400 cursor-not-allowed"
                    : "bg-indigo-600 hover:bg-indigo-700"
                }`}
              >
                {slot}
              </Button>
            ))}
          </div>
        </CardContent>
      </Card>

      {/* Confirmed Bookings Section */}
      <Card className="w-full max-w-3xl shadow-md bg-white border border-gray-200">
        <CardHeader className="bg-gray-100 rounded-t-xl">
          <CardTitle className="text-xl font-semibold text-center text-gray-800">
            Confirmed Bookings
          </CardTitle>
        </CardHeader>
        <CardContent>
          {Object.keys(bookings).length === 0 ? (
            <p className="text-center text-gray-500">No bookings yet.</p>
          ) : (
            <ul className="space-y-4">
              {Object.entries(bookings).map(([key, slots]) => (
                <li key={key} className="border-b pb-2">
                  <p className="font-medium text-gray-700 mb-2">{clinicName(key.split("_")[0])} - {key.split("_")[1]}</p>
                  <div className="space-y-2">
                    {slots.map((b, i) => (
                      <div
                        key={i}
                        className="flex items-center justify-between bg-green-50 p-2 rounded-lg shadow"
                      >
                        <span className="text-gray-800 font-medium">
                          {b.slot} - {b.number} - {b.name} ({b.type === "first" ? "First Consultation" : "Follow-up Consultation"})
                        </span>
                        <Button
                          variant="destructive"
                          size="sm"
                          onClick={() => handleCancel(key, b.slot)}
                        >
                          Cancel
                        </Button>
                      </div>
                    ))}
                  </div>
                </li>
              ))}
            </ul>
          )}
        </CardContent>
      </Card>
    </div>
  );
}
