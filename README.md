//mainwiondow.h

#ifndef MAINWINDOW_H
#define MAINWINDOW_H
#include <QtSerialPort/QSerialPort>
#include <QMainWindow>
#include <QDebug>
#include <QObject>

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
private slots:
    QSerialPort *serial;
    void serialReceived();
    

private:
    Ui::MainWindow *ui;

};
#endif // MAINWINDOW_H
  
  
  
//main.cpp
  
  #include "mainwindow.h"
#include <QApplication>
#include <QDebug>
#include <QTextStream>
#include <QFile>
#include <QtSerialPort/QSerialPort>
#include <QtSerialPort/QSerialPortInfo>
#include <QObject>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    QSerialPort serial;

    serial.setPortName("COM19");

    if(serial.open(QIODevice::ReadWrite)){
        //Now the serial port is open try to set configuration
        if(!serial.setBaudRate(QSerialPort::Baud57600))
            qDebug()<<serial.errorString();

        if(!serial.setDataBits(QSerialPort::Data8))
            qDebug()<<serial.errorString();

        if(!serial.setParity(QSerialPort::NoParity))
            qDebug()<<serial.errorString();

        if(!serial.setStopBits(QSerialPort::OneStop))
            qDebug()<<serial.errorString();

        if(!serial.setFlowControl(QSerialPort::NoFlowControl))
            qDebug()<<serial.errorString();

        //If any error was returned the serial il corrctly configured

        serial.write("M114 \n");
        //the serial must remain opened

        if(serial.waitForReadyRead(100)){
            //Data was returned
            qDebug()<<"Response: "<<serial.readAll();
        }else{
            //No data
            qDebug()<<"Time out";
        }

        //I have finish alla operation
        serial.close();
    }else{
        qDebug()<<"Serial COM19 not opened. Error: "<<serial.errorString();
    }

    MainWindow w;
    w.show();

    return a.exec();
}

  //mainwindow.cpp
  
  #include "mainwindow.h"
#include "ui_mainwindow.h"
#include<QtSerialPort/QSerialPort>
#include <QSerialPort>
#include <QDebug>
#include <QObject>

QSerialPort *serial;
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    serial=new QSerialPort(this);
    serial->setPortName("COM1");
    serial->setBaudRate(QSerialPort::Baud9600);
    serial->setDataBits(QSerialPort::Data8);
    serial->setParity(QSerialPort::NoParity);
    serial->setStopBits(QSerialPort::OneStop);
    serial->setFlowControl(QSerialPort::NoFlowControl);
    serial->open(QIODevice::ReadWrite);
    serial->write("ok*");
    QObject::connect(serial,SIGNAL(readyRead()),this,SLOT(serialReceived()));
}
MainWindow::~MainWindow()
{
    delete ui;
    serial->close();
}
void MainWindow::serialReceived()
{
    QByteArray ba;
    ba = serial->readAll();
    ui -> label->setText(ba);
    qDebug()<<ba;
}






