import sys
import serial
import serial.tools.list_ports
from PyQt5 import QtWidgets,QtGui,QtCore

class serialThreadClass(QtCore.QThread):

    message = QtCore.pyqtSignal(str)
    def __init__(self,parent = None):

        super(serialThreadClass,self).__init__(parent)
        self.serialPort = serial.Serial()
        self.stopflag = False
    def stop(self):
        self.stopflag = True
    def run(self):
        while True:
            if (self.stopflag):
                self.stopflag = False
                break
            elif(self.serialPort.isOpen()):
                try:
                    self.data = self.serialPort.readline()
                except:
                    print("HATA\n")
                self.message.emit(str(self.data.decode()))

                
                
class Pencere(QtWidgets.QWidget):
    def __init__(self):
        super().__init__()
        self.initUi()

    def initUi(self):
        self.portComboBox = QtWidgets.QComboBox()
        self.ports = serial.tools.list_ports.comports()
        for i in self.ports:
            self.portComboBox.addItem(str(i))
        self.baudComboBox = QtWidgets.QComboBox()
        baud = ["300", "1200", "2400", "4800", "9600", "19200", "38400", "57600", "74880", "115200", "230400", "250000",
                "500000", "1000000", "2000000"]
        for i in baud:
            self.baudComboBox.addItem(i)
        self.baudComboBox.setCurrentText(baud[4])

        self.baglan = QtWidgets.QPushButton("Bağlan")
        self.baglantiKes = QtWidgets.QPushButton("Bağlantıyı Kes")
        self.led1on = QtWidgets.QPushButton("LED AÇIK")
        self.led1off = QtWidgets.QPushButton("LED KAPALI")
        self.led2on = QtWidgets.QPushButton("FLASH AÇIK")
        self.led2off = QtWidgets.QPushButton("FLASH KAPALI")

        self.label1 = QtWidgets.QLabel('<font color=red>COM port bağlı değil!!!</font>')

        portVbox = QtWidgets.QVBoxLayout()
        portVbox.addWidget(self.portComboBox)
        portVbox.addWidget(self.baudComboBox)
        portVbox.addWidget(self.baglan)
        portVbox.addWidget(self.baglantiKes)
        portVbox.addWidget(self.label1)

        self.portGroup = QtWidgets.QGroupBox("Port Seçme")
        self.portGroup.setLayout(portVbox)

        buttonHbox1 = QtWidgets.QHBoxLayout()
        buttonHbox1.addWidget(self.led1on)
        buttonHbox1.addWidget(self.led1off)
        buttonHbox2 = QtWidgets.QHBoxLayout()
        buttonHbox2.addWidget(self.led2on)
        buttonHbox2.addWidget(self.led2off)

        self.message = QtWidgets.QTextEdit()
        self.message.setReadOnly(True)
        self.messageTitle = QtWidgets.QLabel("Gelen Mesaj")

        vBox = QtWidgets.QVBoxLayout()
        vBox.addStretch()
        vBox.addWidget(self.portGroup)
        vBox.addLayout(buttonHbox1)
        vBox.addLayout(buttonHbox2)
        vBox.addWidget(self.messageTitle)
        vBox.addWidget(self.message)
        vBox.addStretch()

        hBox = QtWidgets.QHBoxLayout()
        hBox.addStretch()
        hBox.addLayout(vBox)
        hBox.addStretch()


        self.setLayout(hBox)
        self.setWindowTitle(";)")
        self.mySerial = serialThreadClass()
        self.mySerial.message.connect(self.messageTextEdit)
        self.mySerial.start()                

        self.baglan.clicked.connect(self.serialConnect)
        self.baglantiKes.clicked.connect(self.serialDisconnect)
        self.led1on.clicked.connect(lambda: self.leds(self.led1on))
        self.led1off.clicked.connect(lambda: self.leds(self.led1off))
        self.led2on.clicked.connect(lambda: self.leds(self.led2on))
        self.led2off.clicked.connect(lambda: self.leds(self.led2off))


        self.show()
        
    def serialConnect(self):
        self.portText = self.portComboBox.currentText()
        self.port = self.portText.split()
        self.baudrate = self.baudComboBox.currentText()
        self.mySerial.serialPort.baudrate = int(self.baudrate)
        self.mySerial.serialPort.port = self.port[0]
        try:
            self.mySerial.serialPort.open()
        except:
            self.message.append("Bağlantı Hatası!!")
        if(self.mySerial.serialPort.isOpen()):
            self.label1.setText('<font color=green>Bağlandı</font>')
            self.baglan.setEnabled(False)
            self.portComboBox.setEnabled(False)
            self.baudComboBox.setEnabled(False)

    def serialDisconnect(self):
        if self.mySerial.serialPort.isOpen():
            self.mySerial.serialPort.close()
            if self.mySerial.serialPort.isOpen()== False:
                self.label1.setText('<font color=red>Bağlantı Kesildi</font>')
                self.baglan.setEnabled(True)
                self.portComboBox.setEnabled(True)
                self.baudComboBox.setEnabled(True)
        else:
            self.message.append("Seriport Zaten Kapalı.")
    def messageTextEdit(self):
        self.incomingMessage = str(self.mySerial.data.decode())
        self.message.append(self.incomingMessage)
    def leds(self,led):
        if led == self.led1on:
            if self.mySerial.serialPort.isOpen():
                self.mySerial.serialPort.write("1".encode())
            else:
                self.message.append("Seri Port Bağlı Değil.")
        elif led == self.led1off:
            if self.mySerial.serialPort.isOpen():
                self.mySerial.serialPort.write("2".encode())
            else:
                self.message.append("Seri Port Bağlı Değil.")

        if led == self.led2on:
            if self.mySerial.serialPort.isOpen():
                self.mySerial.serialPort.write("3".encode())
            else:
                self.message.append("Seri Port Bağlı Değil.")
        elif led == self.led2off:
            if self.mySerial.serialPort.isOpen():
                self.mySerial.serialPort.write("4".encode())
            else:
                self.message.append("Seri Port Bağlı Değil.")
                
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    pen = Pencere()
    sys.exit(app.exec_())
