/*
 * Copyright (C) 2016 Sony Corporation
 *
 * Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
 * in compliance with the License. You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software distributed under the License
 * is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
 * or implied. See the License for the specific language governing permissions and limitations under
 * the License.
 */
package com.sony.samples.smarttennissensor;

import android.app.AlertDialog;
import android.app.Service;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.content.ComponentName;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.os.RemoteException;
import android.widget.Toast;
import com.sony.smarttennissensor.data.RacketModel;
import com.sony.smarttennissensor.service.Config;
import com.sony.smarttennissensor.service.IHost;
import com.sony.smarttennissensor.service.ILiveModeListener;
import com.sony.smarttennissensor.service.IOnConfigChangeListener;
import com.sony.smarttennissensor.service.IOnSensorChangeListener;
import com.sony.smarttennissensor.service.IOnSensorStateChangeListener;
import com.sony.smarttennissensor.service.IResultListener;
import com.sony.smarttennissensor.service.ISensorConnectListener;
import java.util.Iterator;
import java.util.List;
import java.util.Set;


public class HostDelegate {

    public static final int SENSOR_STATE_DISCONNECTED = 0;
    public static final int SENSOR_STATE_CONNECTING = 1;
    public static final int SENSOR_STATE_CONNECTED = 2;
    public static final int SENSOR_STATE_LIVE_MODE = 3;

    private IHost mCallback = null;
    private List<RacketModel> mRacketList = null;
    private Context context = null;

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Toast.makeText(context, "OnServiceConnected", Toast.LENGTH_LONG).show();
            mCallback = IHost.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Toast.makeText(context, "OnService DIS Connected", Toast.LENGTH_LONG).show();
        }
    };

    public void setContext(Context context) {
        if (context == null) {
            throw new NullPointerException("Argument cannot be null");
        }
        this.context = context;
    }

    public void bind() {

        Intent intent = new Intent();
        ComponentName cn = new ComponentName("com.sony.smarttennissensor.sdk.host", "com.sony.smarttennissensor.service.AriakeService");
        intent.setComponent(cn);

        context.bindService(intent, mConnection, Service.BIND_AUTO_CREATE);

    }


    public String getSensorState() {

        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return null;
        }

        String statusMessage;
        try {
            int state = mCallback.getSensorState();
            switch (state) {
                case SENSOR_STATE_DISCONNECTED:
                    statusMessage = "SENSOR DISCONNECTED";
                    break;
                case SENSOR_STATE_CONNECTING:
                    statusMessage = "SENSOR CONNECTING";
                    break;
                case SENSOR_STATE_CONNECTED:
                    statusMessage = "SENSOR CONNECTED";
                    break;
                case SENSOR_STATE_LIVE_MODE:
                    statusMessage = "LIVE MODE";
                    break;
                default:
                    statusMessage = "SOME OTHER STATE:" + state;
            }

        } catch (RemoteException e) {
            e.printStackTrace();
            statusMessage = "getSensorState returned Exception= " + e.getMessage();
        } catch (NullPointerException n) {
            n.printStackTrace();
            statusMessage = "getSensorState returned Exception= " + n.getMessage();
        }

        return statusMessage;

    }


    public void promptUserAndConnectToSensor(final ISensorConnectListener connectListener,
                                             final IOnSensorChangeListener changeListener,
                                             final IOnSensorStateChangeListener stateChangeListener) {
        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
        final Set<BluetoothDevice> pairedDevices = mBluetoothAdapter.getBondedDevices();
        final String[] btNames = new String[pairedDevices.size()];

        int i = 0;

        for (BluetoothDevice bt : pairedDevices) {
            btNames[i++] = bt.getName();
        }

        AlertDialog.Builder builder = new AlertDialog.Builder(context);
        builder.setTitle("Select device to connect with");
        builder.setNegativeButton("Cancel",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });


        builder.setItems(btNames, new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int item) {
                try {

                    mCallback.connect((BluetoothDevice) pairedDevices.toArray()[item],
                            connectListener, changeListener, stateChangeListener);

                } catch (RemoteException r) {
                    r.printStackTrace();
                }

            }
        });

        AlertDialog alertDialog = builder.create();
        alertDialog.show();

    }


    public void connectToFirstPairedSensor(final ISensorConnectListener connectListener,
                                           final IOnSensorChangeListener changeListener,
                                           final IOnSensorStateChangeListener sensorStateChangeListener) {

        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
        final Set<BluetoothDevice> pairedDevices = mBluetoothAdapter.getBondedDevices();
        Iterator<BluetoothDevice> i = pairedDevices.iterator();
        if (i.hasNext()) {
            try {
                BluetoothDevice device = i.next();
                checkSensorStateBeforeConnecting(device, connectListener, changeListener, sensorStateChangeListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }


    }

    private void checkSensorStateBeforeConnecting(BluetoothDevice device, ISensorConnectListener connectListener,
                                                  IOnSensorChangeListener changeListener,
                                                  IOnSensorStateChangeListener stateChangeListener) throws RemoteException {

        try {
            mCallback.connect(device, connectListener, changeListener, stateChangeListener);
        } catch (NullPointerException n) {
            n.printStackTrace();
            bind(); //rebinding
        }

    }

    public void disconnect() {

        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        try {
            mCallback.disconnect();
        } catch (RemoteException r) {
            r.printStackTrace();
        }
    }

    public List<RacketModel> getRacketList() {

        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return null;
        }

        try {
            return mCallback.getRacketList();
        } catch (RemoteException r) {
            r.printStackTrace();
        }
        return null;
    }

    public Config getConfig() {

        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return null;
        }

        Config config = null;
        try {
            config = mCallback.getConfig();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
        return config;

    }


    public void setConfig(Config config, IResultListener resultListener, IOnConfigChangeListener
            changeListener) {
        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        try {
            if (config == null && mRacketList != null && mRacketList.size() > 0) {
                RacketModel racket = mRacketList.get(0);
                config = new Config(racket.getId(), Config.HAND_LEFT);
            }

            mCallback.setConfig(config, resultListener, changeListener);
        } catch (RemoteException r) {
            r.printStackTrace();
        }

    }


    public void startLiveMode(ILiveModeListener listener) {
        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        try {
            mCallback.startLiveMode(listener);
        } catch (RemoteException e) {
            e.printStackTrace();
        }

    }

    public void stopLiveMode() {
        if (mCallback == null) {
            Toast.makeText(context, "Please bind first.", Toast.LENGTH_SHORT).show();
            return;
        }

        try {
            mCallback.stopLiveMode();
        } catch (RemoteException e) {
            e.printStackTrace();
        }

    }


}
