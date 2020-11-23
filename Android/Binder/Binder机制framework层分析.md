---
title: Binder机制framework层分析
date: 2020-05-20 13:42:10
tags: 
	- Android
	- Binder
categories: 
	- Android
	- Binder
---

# 相关源码

基于6.0.0_r1地址[源码地址](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.0_r1/core/java/android/os/)

```
framework/base/core/java/android/os/
  - IInterface.java
  - IServiceManager.java
  - ServiceManager.java
  - ServiceManagerNative.java(内含ServiceManagerProxy类)

framework/base/core/java/android/os/
  - IBinder.java
  - Binder.java(内含BinderProxy类)
  - Parcel.java

framework/base/core/java/com/android/internal/os/
  - BinderInternal.java

framework/base/core/jni/
  - AndroidRuntime.cpp
  - android_os_Parcel.cpp
  - android_util_Binder.cpp
```

**IInterface**

```
/**
 * Base class for Binder interfaces.  When defining a new interface,
 * you must derive it from IInterface.
 */
public interface IInterface
{
    /**
     * Retrieve the Binder object associated with this interface.
     * You must use this instead of a plain cast, so that proxy objects
     * can return the correct result.
     */
    public IBinder asBinder();
}
```

**IServiceManager**

```
/**
 * Basic interface for finding and publishing system services.
 * 
 * An implementation of this interface is usually published as the
 * global context object, which can be retrieved via
 * BinderNative.getContextObject().  An easy way to retrieve this
 * is with the static method BnServiceManager.getDefault().
 * 
 * @hide
 */
public interface IServiceManager extends IInterface
{
    /**
     * Retrieve an existing service called @a name from the
     * service manager.  Blocks for a few seconds waiting for it to be
     * published if it does not already exist.
     */
    public IBinder getService(String name) throws RemoteException;
    
    /**
     * Retrieve an existing service called @a name from the
     * service manager.  Non-blocking.
     */
    public IBinder checkService(String name) throws RemoteException;
    /**
     * Place a new @a service called @a name into the service
     * manager.
     */
    public void addService(String name, IBinder service, boolean allowIsolated)
                throws RemoteException;
    /**
     * Return a list of all currently running services.
     */
    public String[] listServices() throws RemoteException;
    /**
     * Assign a permission controller to the service manager.  After set, this
     * interface is checked before any services are added.
     */
    public void setPermissionController(IPermissionController controller)
            throws RemoteException;
    
    static final String descriptor = "android.os.IServiceManager";
    int GET_SERVICE_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION;
    int CHECK_SERVICE_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION+1;
    int ADD_SERVICE_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION+2;
    int LIST_SERVICES_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION+3;
    int CHECK_SERVICES_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION+4;
    int SET_PERMISSION_CONTROLLER_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION+5;
}
```



**ServiceManager**

```
/** @hide */
public final class ServiceManager {
    private static final String TAG = "ServiceManager";
    private static IServiceManager sServiceManager;
    private static HashMap<String, IBinder> sCache = new HashMap<String, IBinder>();
    private static IServiceManager getIServiceManager() {
        if (sServiceManager != null) {
            return sServiceManager;
        }
        // Find the service manager
        sServiceManager = ServiceManagerNative.asInterface(BinderInternal.getContextObject());
        return sServiceManager;
    }
    /**
     * Returns a reference to a service with the given name.
     * 
     * @param name the name of the service to get
     * @return a reference to the service, or <code>null</code> if the service doesn't exist
     */
    public static IBinder getService(String name) {
        try {
            IBinder service = sCache.get(name);
            if (service != null) {
                return service;
            } else {
                return getIServiceManager().getService(name);
            }
        } catch (RemoteException e) {
            Log.e(TAG, "error in getService", e);
        }
        return null;
    }
    /**
     * Place a new @a service called @a name into the service
     * manager.
     * 
     * @param name the name of the new service
     * @param service the service object
     */
    public static void addService(String name, IBinder service) {
        try {
            getIServiceManager().addService(name, service, false);
        } catch (RemoteException e) {
            Log.e(TAG, "error in addService", e);
        }
    }
    /**
     * Place a new @a service called @a name into the service
     * manager.
     * 
     * @param name the name of the new service
     * @param service the service object
     * @param allowIsolated set to true to allow isolated sandboxed processes
     * to access this service
     */
    public static void addService(String name, IBinder service, boolean allowIsolated) {
        try {
            getIServiceManager().addService(name, service, allowIsolated);
        } catch (RemoteException e) {
            Log.e(TAG, "error in addService", e);
        }
    }
    
    /**
     * Retrieve an existing service called @a name from the
     * service manager.  Non-blocking.
     */
    public static IBinder checkService(String name) {
        try {
            IBinder service = sCache.get(name);
            if (service != null) {
                return service;
            } else {
                return getIServiceManager().checkService(name);
            }
        } catch (RemoteException e) {
            Log.e(TAG, "error in checkService", e);
            return null;
        }
    }
    /**
     * Return a list of all currently running services.
     */
    public static String[] listServices() throws RemoteException {
        try {
            return getIServiceManager().listServices();
        } catch (RemoteException e) {
            Log.e(TAG, "error in listServices", e);
            return null;
        }
    }
    /**
     * This is only intended to be called when the process is first being brought
     * up and bound by the activity manager. There is only one thread in the process
     * at that time, so no locking is done.
     * 
     * @param cache the cache of service references
     * @hide
     */
    public static void initServiceCache(Map<String, IBinder> cache) {
        if (sCache.size() != 0) {
            throw new IllegalStateException("setServiceCache may only be called once");
        }
        sCache.putAll(cache);
    }
}
```

**ServiceManagerNative**

```
/**
 * Native implementation of the service manager.  Most clients will only
 * care about getDefault() and possibly asInterface().
 * @hide
 */
public abstract class ServiceManagerNative extends Binder implements IServiceManager
{
    /**
     * Cast a Binder object into a service manager interface, generating
     * a proxy if needed.
     */
    static public IServiceManager asInterface(IBinder obj)
    {
        if (obj == null) {
            return null;
        }
        IServiceManager in =
            (IServiceManager)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }
        
        return new ServiceManagerProxy(obj);
    }
    
    public ServiceManagerNative()
    {
        attachInterface(this, descriptor);
    }
    
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
    {
        try {
            switch (code) {
            case IServiceManager.GET_SERVICE_TRANSACTION: {
                data.enforceInterface(IServiceManager.descriptor);
                String name = data.readString();
                IBinder service = getService(name);
                reply.writeStrongBinder(service);
                return true;
            }
    
            case IServiceManager.CHECK_SERVICE_TRANSACTION: {
                data.enforceInterface(IServiceManager.descriptor);
                String name = data.readString();
                IBinder service = checkService(name);
                reply.writeStrongBinder(service);
                return true;
            }
    
            case IServiceManager.ADD_SERVICE_TRANSACTION: {
                data.enforceInterface(IServiceManager.descriptor);
                String name = data.readString();
                IBinder service = data.readStrongBinder();
                boolean allowIsolated = data.readInt() != 0;
                addService(name, service, allowIsolated);
                return true;
            }
    
            case IServiceManager.LIST_SERVICES_TRANSACTION: {
                data.enforceInterface(IServiceManager.descriptor);
                String[] list = listServices();
                reply.writeStringArray(list);
                return true;
            }
            
            case IServiceManager.SET_PERMISSION_CONTROLLER_TRANSACTION: {
                data.enforceInterface(IServiceManager.descriptor);
                IPermissionController controller
                        = IPermissionController.Stub.asInterface(
                                data.readStrongBinder());
                setPermissionController(controller);
                return true;
            }
            }
        } catch (RemoteException e) {
        }
        
        return false;
    }
    public IBinder asBinder()
    {
        return this;
    }
}
class ServiceManagerProxy implements IServiceManager {
    public ServiceManagerProxy(IBinder remote) {
        mRemote = remote;
    }
    
    public IBinder asBinder() {
        return mRemote;
    }
    
    public IBinder getService(String name) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeString(name);
        mRemote.transact(GET_SERVICE_TRANSACTION, data, reply, 0);
        IBinder binder = reply.readStrongBinder();
        reply.recycle();
        data.recycle();
        return binder;
    }
    public IBinder checkService(String name) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeString(name);
        mRemote.transact(CHECK_SERVICE_TRANSACTION, data, reply, 0);
        IBinder binder = reply.readStrongBinder();
        reply.recycle();
        data.recycle();
        return binder;
    }
    public void addService(String name, IBinder service, boolean allowIsolated)
            throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeString(name);
        data.writeStrongBinder(service);
        data.writeInt(allowIsolated ? 1 : 0);
        mRemote.transact(ADD_SERVICE_TRANSACTION, data, reply, 0);
        reply.recycle();
        data.recycle();
    }
    
    public String[] listServices() throws RemoteException {
        ArrayList<String> services = new ArrayList<String>();
        int n = 0;
        while (true) {
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();
            data.writeInterfaceToken(IServiceManager.descriptor);
            data.writeInt(n);
            n++;
            try {
                boolean res = mRemote.transact(LIST_SERVICES_TRANSACTION, data, reply, 0);
                if (!res) {
                    break;
                }
            } catch (RuntimeException e) {
                // The result code that is returned by the C++ code can
                // cause the call to throw an exception back instead of
                // returning a nice result...  so eat it here and go on.
                break;
            }
            services.add(reply.readString());
            reply.recycle();
            data.recycle();
        }
        String[] array = new String[services.size()];
        services.toArray(array);
        return array;
    }
    public void setPermissionController(IPermissionController controller)
            throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeStrongBinder(controller.asBinder());
        mRemote.transact(SET_PERMISSION_CONTROLLER_TRANSACTION, data, reply, 0);
        reply.recycle();
        data.recycle();
    }
    private IBinder mRemote;
}
```

**IBinder**

```
public interface IBinder {
    /**
     * The first transaction code available for user commands.
     */
    int FIRST_CALL_TRANSACTION  = 0x00000001;
    /**
     * The last transaction code available for user commands.
     */
    int LAST_CALL_TRANSACTION   = 0x00ffffff;
    
    /**
     * IBinder protocol transaction code: pingBinder().
     */
    int PING_TRANSACTION        = ('_'<<24)|('P'<<16)|('N'<<8)|'G';
    
    /**
     * IBinder protocol transaction code: dump internal state.
     */
    int DUMP_TRANSACTION        = ('_'<<24)|('D'<<16)|('M'<<8)|'P';
    
    /**
     * IBinder protocol transaction code: interrogate the recipient side
     * of the transaction for its canonical interface descriptor.
     */
    int INTERFACE_TRANSACTION   = ('_'<<24)|('N'<<16)|('T'<<8)|'F';
    /**
     * IBinder protocol transaction code: send a tweet to the target
     * object.  The data in the parcel is intended to be delivered to
     * a shared messaging service associated with the object; it can be
     * anything, as long as it is not more than 130 UTF-8 characters to
     * conservatively fit within common messaging services.  As part of
     * {@link Build.VERSION_CODES#HONEYCOMB_MR2}, all Binder objects are
     * expected to support this protocol for fully integrated tweeting
     * across the platform.  To support older code, the default implementation
     * logs the tweet to the main log as a simple emulation of broadcasting
     * it publicly over the Internet.
     * 
     * <p>Also, upon completing the dispatch, the object must make a cup
     * of tea, return it to the caller, and exclaim "jolly good message
     * old boy!".
     */
    int TWEET_TRANSACTION   = ('_'<<24)|('T'<<16)|('W'<<8)|'T';
    /**
     * IBinder protocol transaction code: tell an app asynchronously that the
     * caller likes it.  The app is responsible for incrementing and maintaining
     * its own like counter, and may display this value to the user to indicate the
     * quality of the app.  This is an optional command that applications do not
     * need to handle, so the default implementation is to do nothing.
     * 
     * <p>There is no response returned and nothing about the
     * system will be functionally affected by it, but it will improve the
     * app's self-esteem.
     */
    int LIKE_TRANSACTION   = ('_'<<24)|('L'<<16)|('I'<<8)|'K';
    /** @hide */
    int SYSPROPS_TRANSACTION = ('_'<<24)|('S'<<16)|('P'<<8)|'R';
    /**
     * Flag to {@link #transact}: this is a one-way call, meaning that the
     * caller returns immediately, without waiting for a result from the
     * callee. Applies only if the caller and callee are in different
     * processes.
     */
    int FLAG_ONEWAY             = 0x00000001;
    /**
     * Limit that should be placed on IPC sizes to keep them safely under the
     * transaction buffer limit.
     * @hide
     */
    public static final int MAX_IPC_SIZE = 64 * 1024;
    /**
     * Get the canonical name of the interface supported by this binder.
     */
    public String getInterfaceDescriptor() throws RemoteException;
    /**
     * Check to see if the object still exists.
     * 
     * @return Returns false if the
     * hosting process is gone, otherwise the result (always by default
     * true) returned by the pingBinder() implementation on the other
     * side.
     */
    public boolean pingBinder();
    /**
     * Check to see if the process that the binder is in is still alive.
     *
     * @return false if the process is not alive.  Note that if it returns
     * true, the process may have died while the call is returning.
     */
    public boolean isBinderAlive();
    
    /**
     * Attempt to retrieve a local implementation of an interface
     * for this Binder object.  If null is returned, you will need
     * to instantiate a proxy class to marshall calls through
     * the transact() method.
     */
    public IInterface queryLocalInterface(String descriptor);
    
    /**
     * Print the object's state into the given stream.
     * 
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param args additional arguments to the dump request.
     */
    public void dump(FileDescriptor fd, String[] args) throws RemoteException;
    
    /**
     * Like {@link #dump(FileDescriptor, String[])} but always executes
     * asynchronously.  If the object is local, a new thread is created
     * to perform the dump.
     *
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param args additional arguments to the dump request.
     */
    public void dumpAsync(FileDescriptor fd, String[] args) throws RemoteException;
    /**
     * Perform a generic operation with the object.
     * 
     * @param code The action to perform.  This should
     * be a number between {@link #FIRST_CALL_TRANSACTION} and
     * {@link #LAST_CALL_TRANSACTION}.
     * @param data Marshalled data to send to the target.  Must not be null.
     * If you are not sending any data, you must create an empty Parcel
     * that is given here.
     * @param reply Marshalled data to be received from the target.  May be
     * null if you are not interested in the return value.
     * @param flags Additional operation flags.  Either 0 for a normal
     * RPC, or {@link #FLAG_ONEWAY} for a one-way RPC.
     */
    public boolean transact(int code, Parcel data, Parcel reply, int flags)
        throws RemoteException;
    /**
     * Interface for receiving a callback when the process hosting an IBinder
     * has gone away.
     * 
     * @see #linkToDeath
     */
    public interface DeathRecipient {
        public void binderDied();
    }
    /**
     * Register the recipient for a notification if this binder
     * goes away.  If this binder object unexpectedly goes away
     * (typically because its hosting process has been killed),
     * then the given {@link DeathRecipient}'s
     * {@link DeathRecipient#binderDied DeathRecipient.binderDied()} method
     * will be called.
     * 
     * <p>You will only receive death notifications for remote binders,
     * as local binders by definition can't die without you dying as well.
     * 
     * @throws RemoteException if the target IBinder's
     * process has already died.
     * 
     * @see #unlinkToDeath
     */
    public void linkToDeath(DeathRecipient recipient, int flags)
            throws RemoteException;
    /**
     * Remove a previously registered death notification.
     * The recipient will no longer be called if this object
     * dies.
     * 
     * @return {@code true} if the <var>recipient</var> is successfully
     * unlinked, assuring you that its
     * {@link DeathRecipient#binderDied DeathRecipient.binderDied()} method
     * will not be called;  {@code false} if the target IBinder has already
     * died, meaning the method has been (or soon will be) called.
     * 
     * @throws java.util.NoSuchElementException if the given
     * <var>recipient</var> has not been registered with the IBinder, and
     * the IBinder is still alive.  Note that if the <var>recipient</var>
     * was never registered, but the IBinder has already died, then this
     * exception will <em>not</em> be thrown, and you will receive a false
     * return value instead.
     */
    public boolean unlinkToDeath(DeathRecipient recipient, int flags);
}
```

**Binder**

BinderProxy是Binder的内部类实现IBinder接口

```
public class Binder implements IBinder {
    /*
     * Set this flag to true to detect anonymous, local or member classes
     * that extend this Binder class and that are not static. These kind
     * of classes can potentially create leaks.
     */
    private static final boolean FIND_POTENTIAL_LEAKS = false;
    private static final boolean CHECK_PARCEL_SIZE = false;
    static final String TAG = "Binder";
    /**
     * Control whether dump() calls are allowed.
     */
    private static String sDumpDisabled = null;
    /* mObject is used by native code, do not remove or rename */
    private long mObject;
    private IInterface mOwner;
    private String mDescriptor;
    
    /**
     * Return the ID of the process that sent you the current transaction
     * that is being processed.  This pid can be used with higher-level
     * system services to determine its identity and check permissions.
     * If the current thread is not currently executing an incoming transaction,
     * then its own pid is returned.
     */
    public static final native int getCallingPid();
    
    /**
     * Return the Linux uid assigned to the process that sent you the
     * current transaction that is being processed.  This uid can be used with
     * higher-level system services to determine its identity and check
     * permissions.  If the current thread is not currently executing an
     * incoming transaction, then its own uid is returned.
     */
    public static final native int getCallingUid();
    /**
     * Return the UserHandle assigned to the process that sent you the
     * current transaction that is being processed.  This is the user
     * of the caller.  It is distinct from {@link #getCallingUid()} in that a
     * particular user will have multiple distinct apps running under it each
     * with their own uid.  If the current thread is not currently executing an
     * incoming transaction, then its own UserHandle is returned.
     */
    public static final UserHandle getCallingUserHandle() {
        return new UserHandle(UserHandle.getUserId(getCallingUid()));
    }
    /**
     * Reset the identity of the incoming IPC on the current thread.  This can
     * be useful if, while handling an incoming call, you will be calling
     * on interfaces of other objects that may be local to your process and
     * need to do permission checks on the calls coming into them (so they
     * will check the permission of your own local process, and not whatever
     * process originally called you).
     *
     * @return Returns an opaque token that can be used to restore the
     * original calling identity by passing it to
     * {@link #restoreCallingIdentity(long)}.
     *
     * @see #getCallingPid()
     * @see #getCallingUid()
     * @see #restoreCallingIdentity(long)
     */
    public static final native long clearCallingIdentity();
    /**
     * Restore the identity of the incoming IPC on the current thread
     * back to a previously identity that was returned by {@link
     * #clearCallingIdentity}.
     *
     * @param token The opaque token that was previously returned by
     * {@link #clearCallingIdentity}.
     *
     * @see #clearCallingIdentity
     */
    public static final native void restoreCallingIdentity(long token);
    /**
     * Sets the native thread-local StrictMode policy mask.
     *
     * <p>The StrictMode settings are kept in two places: a Java-level
     * threadlocal for libcore/Dalvik, and a native threadlocal (set
     * here) for propagation via Binder calls.  This is a little
     * unfortunate, but necessary to break otherwise more unfortunate
     * dependencies either of Dalvik on Android, or Android
     * native-only code on Dalvik.
     *
     * @see StrictMode
     * @hide
     */
    public static final native void setThreadStrictModePolicy(int policyMask);
    /**
     * Gets the current native thread-local StrictMode policy mask.
     *
     * @see #setThreadStrictModePolicy
     * @hide
     */
    public static final native int getThreadStrictModePolicy();
    /**
     * Flush any Binder commands pending in the current thread to the kernel
     * driver.  This can be
     * useful to call before performing an operation that may block for a long
     * time, to ensure that any pending object references have been released
     * in order to prevent the process from holding on to objects longer than
     * it needs to.
     */
    public static final native void flushPendingCommands();
    
    /**
     * Add the calling thread to the IPC thread pool.  This function does
     * not return until the current process is exiting.
     */
    public static final native void joinThreadPool();
    /**
     * Returns true if the specified interface is a proxy.
     * @hide
     */
    public static final boolean isProxy(IInterface iface) {
        return iface.asBinder() != iface;
    }
    /**
     * Call blocks until the number of executing binder threads is less
     * than the maximum number of binder threads allowed for this process.
     * @hide
     */
    public static final native void blockUntilThreadAvailable();
    /**
     * Default constructor initializes the object.
     */
    public Binder() {
        init();
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Binder> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Binder class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
    }
    
    /**
     * Convenience method for associating a specific interface with the Binder.
     * After calling, queryLocalInterface() will be implemented for you
     * to return the given owner IInterface when the corresponding
     * descriptor is requested.
     */
    public void attachInterface(IInterface owner, String descriptor) {
        mOwner = owner;
        mDescriptor = descriptor;
    }
    
    /**
     * Default implementation returns an empty interface name.
     */
    public String getInterfaceDescriptor() {
        return mDescriptor;
    }
    /**
     * Default implementation always returns true -- if you got here,
     * the object is alive.
     */
    public boolean pingBinder() {
        return true;
    }
    /**
     * {@inheritDoc}
     *
     * Note that if you're calling on a local binder, this always returns true
     * because your process is alive if you're calling it.
     */
    public boolean isBinderAlive() {
        return true;
    }
    
    /**
     * Use information supplied to attachInterface() to return the
     * associated IInterface if it matches the requested
     * descriptor.
     */
    public IInterface queryLocalInterface(String descriptor) {
        if (mDescriptor.equals(descriptor)) {
            return mOwner;
        }
        return null;
    }
    /**
     * Control disabling of dump calls in this process.  This is used by the system
     * process watchdog to disable incoming dump calls while it has detecting the system
     * is hung and is reporting that back to the activity controller.  This is to
     * prevent the controller from getting hung up on bug reports at this point.
     * @hide
     *
     * @param msg The message to show instead of the dump; if null, dumps are
     * re-enabled.
     */
    public static void setDumpDisabled(String msg) {
        synchronized (Binder.class) {
            sDumpDisabled = msg;
        }
    }
    /**
     * Default implementation is a stub that returns false.  You will want
     * to override this to do the appropriate unmarshalling of transactions.
     *
     * <p>If you want to call this, call transact().
     */
    protected boolean onTransact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (code == INTERFACE_TRANSACTION) {
            reply.writeString(getInterfaceDescriptor());
            return true;
        } else if (code == DUMP_TRANSACTION) {
            ParcelFileDescriptor fd = data.readFileDescriptor();
            String[] args = data.readStringArray();
            if (fd != null) {
                try {
                    dump(fd.getFileDescriptor(), args);
                } finally {
                    try {
                        fd.close();
                    } catch (IOException e) {
                        // swallowed, not propagated back to the caller
                    }
                }
            }
            // Write the StrictMode header.
            if (reply != null) {
                reply.writeNoException();
            } else {
                StrictMode.clearGatheredViolations();
            }
            return true;
        }
        return false;
    }
    /**
     * Implemented to call the more convenient version
     * {@link #dump(FileDescriptor, PrintWriter, String[])}.
     */
    public void dump(FileDescriptor fd, String[] args) {
        FileOutputStream fout = new FileOutputStream(fd);
        PrintWriter pw = new FastPrintWriter(fout);
        try {
            final String disabled;
            synchronized (Binder.class) {
                disabled = sDumpDisabled;
            }
            if (disabled == null) {
                try {
                    dump(fd, pw, args);
                } catch (SecurityException e) {
                    pw.println("Security exception: " + e.getMessage());
                    throw e;
                } catch (Throwable e) {
                    // Unlike usual calls, in this case if an exception gets thrown
                    // back to us we want to print it back in to the dump data, since
                    // that is where the caller expects all interesting information to
                    // go.
                    pw.println();
                    pw.println("Exception occurred while dumping:");
                    e.printStackTrace(pw);
                }
            } else {
                pw.println(sDumpDisabled);
            }
        } finally {
            pw.flush();
        }
    }
    
    /**
     * Like {@link #dump(FileDescriptor, String[])}, but ensures the target
     * executes asynchronously.
     */
    public void dumpAsync(final FileDescriptor fd, final String[] args) {
        final FileOutputStream fout = new FileOutputStream(fd);
        final PrintWriter pw = new FastPrintWriter(fout);
        Thread thr = new Thread("Binder.dumpAsync") {
            public void run() {
                try {
                    dump(fd, pw, args);
                } finally {
                    pw.flush();
                }
            }
        };
        thr.start();
    }
    /**
     * Print the object's state into the given stream.
     * 
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param fout The file to which you should dump your state.  This will be
     * closed for you after you return.
     * @param args additional arguments to the dump request.
     */
    protected void dump(FileDescriptor fd, PrintWriter fout, String[] args) {
    }
    /**
     * Default implementation rewinds the parcels and calls onTransact.  On
     * the remote side, transact calls into the binder to do the IPC.
     */
    public final boolean transact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (false) Log.v("Binder", "Transact: " + code + " to " + this);
        if (data != null) {
            data.setDataPosition(0);
        }
        boolean r = onTransact(code, data, reply, flags);
        if (reply != null) {
            reply.setDataPosition(0);
        }
        return r;
    }
    
    /**
     * Local implementation is a no-op.
     */
    public void linkToDeath(DeathRecipient recipient, int flags) {
    }
    /**
     * Local implementation is a no-op.
     */
    public boolean unlinkToDeath(DeathRecipient recipient, int flags) {
        return true;
    }
    
    protected void finalize() throws Throwable {
        try {
            destroy();
        } finally {
            super.finalize();
        }
    }
    static void checkParcel(IBinder obj, int code, Parcel parcel, String msg) {
        if (CHECK_PARCEL_SIZE && parcel.dataSize() >= 800*1024) {
            // Trying to send > 800k, this is way too much
            StringBuilder sb = new StringBuilder();
            sb.append(msg);
            sb.append(": on ");
            sb.append(obj);
            sb.append(" calling ");
            sb.append(code);
            sb.append(" size ");
            sb.append(parcel.dataSize());
            sb.append(" (data: ");
            parcel.setDataPosition(0);
            sb.append(parcel.readInt());
            sb.append(", ");
            sb.append(parcel.readInt());
            sb.append(", ");
            sb.append(parcel.readInt());
            sb.append(")");
            Slog.wtfStack(TAG, sb.toString());
        }
    }
    private native final void init();
    private native final void destroy();
    // Entry point from android_util_Binder.cpp's onTransact
    private boolean execTransact(int code, long dataObj, long replyObj,
            int flags) {
        Parcel data = Parcel.obtain(dataObj);
        Parcel reply = Parcel.obtain(replyObj);
        // theoretically, we should call transact, which will call onTransact,
        // but all that does is rewind it, and we just got these from an IPC,
        // so we'll just call it directly.
        boolean res;
        // Log any exceptions as warnings, don't silently suppress them.
        // If the call was FLAG_ONEWAY then these exceptions disappear into the ether.
        try {
            res = onTransact(code, data, reply, flags);
        } catch (RemoteException e) {
            if ((flags & FLAG_ONEWAY) != 0) {
                Log.w(TAG, "Binder call failed.", e);
            } else {
                reply.setDataPosition(0);
                reply.writeException(e);
            }
            res = true;
        } catch (RuntimeException e) {
            if ((flags & FLAG_ONEWAY) != 0) {
                Log.w(TAG, "Caught a RuntimeException from the binder stub implementation.", e);
            } else {
                reply.setDataPosition(0);
                reply.writeException(e);
            }
            res = true;
        } catch (OutOfMemoryError e) {
            // Unconditionally log this, since this is generally unrecoverable.
            Log.e(TAG, "Caught an OutOfMemoryError from the binder stub implementation.", e);
            RuntimeException re = new RuntimeException("Out of memory", e);
            reply.setDataPosition(0);
            reply.writeException(re);
            res = true;
        }
        checkParcel(this, code, reply, "Unreasonably large binder reply buffer");
        reply.recycle();
        data.recycle();
        // Just in case -- we are done with the IPC, so there should be no more strict
        // mode violations that have gathered for this thread.  Either they have been
        // parceled and are now in transport off to the caller, or we are returning back
        // to the main transaction loop to wait for another incoming transaction.  Either
        // way, strict mode begone!
        StrictMode.clearGatheredViolations();
        return res;
    }
}
final class BinderProxy implements IBinder {
    public native boolean pingBinder();
    public native boolean isBinderAlive();
    public IInterface queryLocalInterface(String descriptor) {
        return null;
    }
    public boolean transact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        Binder.checkParcel(this, code, data, "Unreasonably large binder buffer");
        return transactNative(code, data, reply, flags);
    }
    public native String getInterfaceDescriptor() throws RemoteException;
    public native boolean transactNative(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException;
    public native void linkToDeath(DeathRecipient recipient, int flags)
            throws RemoteException;
    public native boolean unlinkToDeath(DeathRecipient recipient, int flags);
    public void dump(FileDescriptor fd, String[] args) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeFileDescriptor(fd);
        data.writeStringArray(args);
        try {
            transact(DUMP_TRANSACTION, data, reply, 0);
            reply.readException();
        } finally {
            data.recycle();
            reply.recycle();
        }
    }
    
    public void dumpAsync(FileDescriptor fd, String[] args) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeFileDescriptor(fd);
        data.writeStringArray(args);
        try {
            transact(DUMP_TRANSACTION, data, reply, FLAG_ONEWAY);
        } finally {
            data.recycle();
            reply.recycle();
        }
    }
    BinderProxy() {
        mSelf = new WeakReference(this);
    }
    
    @Override
    protected void finalize() throws Throwable {
        try {
            destroy();
        } finally {
            super.finalize();
        }
    }
    
    private native final void destroy();
    
    private static final void sendDeathNotice(DeathRecipient recipient) {
        if (false) Log.v("JavaBinder", "sendDeathNotice to " + recipient);
        try {
            recipient.binderDied();
        }
        catch (RuntimeException exc) {
            Log.w("BinderNative", "Uncaught exception from death notification",
                    exc);
        }
    }
    
    final private WeakReference mSelf;
    private long mObject;
    private long mOrgue;
}
```

# Binder概述

## Binder架构

binder在framework层，采用JNI技术来调用native(C/C++)层的binder架构，从而为上层应用程序提供服务。 看过binder系列之前的文章，我们知道native层中，binder是C/S架构，分为Bn端(Server)和Bp端(Client)。对于java层在命名与架构上非常相近，同样实现了一套IPC通信架构。

![java_binder](http://gityuan.com/images/binder/java_binder/java_binder.jpg)

- 图中红色代表整个framework层 binder架构相关组件；
  - Binder类代表Server端，BinderProxy类代码Client端；
- 图中蓝色代表Native层Binder架构相关组件；
- 上层framework层的Binder逻辑是建立在Native层架构基础之上的，核心逻辑都是交予Native层方法来处理。
- framework层的ServiceManager类与Native层的功能并不完全对应，framework层的ServiceManager类的实现最终是通过BinderProxy传递给Native层来完成的，后面会详细说明。

## Binder类图

![class_java_binder](http://gityuan.com/images/binder/java_binder/class_ServiceManager.jpg)

- **ServiceManager：**通过getIServiceManager方法获取的是ServiceManagerProxy对象； ServiceManager的addService, getService实际工作都交由ServiceManagerProxy的相应方法来处理；
- **ServiceManagerProxy：**其成员变量mRemote指向BinderProxy对象，ServiceManagerProxy的addService, getService方法最终是交由mRemote来完成。
- **ServiceManagerNative**：其方法asInterface()返回的是ServiceManagerProxy对象，ServiceManager便是借助ServiceManagerNative类来找到ServiceManagerProxy；
- **Binder：**其成员变量mObject和方法execTransact()用于native方法
- **BinderInternal：**内部有一个GcWatcher类，用于处理和调试与Binder相关的垃圾回收。
- **IBinder：**接口中常量FLAG_ONEWAY：客户端利用binder跟服务端通信是阻塞式的，但如果设置了FLAG_ONEWAY，这成为非阻塞的调用方式，客户端能立即返回，服务端采用回调方式来通知客户端完成情况。另外IBinder接口有一个内部接口DeathDecipient(死亡通告)。

## Binder类分层

整个Binder从kernel至，native，JNI，Framework层所涉及的全部类

![java_binder_framework](http://gityuan.com/images/binder/java_binder_framework.jpg)

# 注册服务

##  SM.addService

```
public static void addService(String name, IBinder service) {
        try {
            getIServiceManager().addService(name, service, false);
        } catch (RemoteException e) {
            Log.e(TAG, "error in addService", e);
        }
    }
```

先来看看getIServiceManager()过程，如下：

## getIServiceManager

**ServiceManager.java**

```
private static IServiceManager getIServiceManager() {
        if (sServiceManager != null) {
            return sServiceManager;
        }
        // Find the service manager
        sServiceManager = ServiceManagerNative.asInterface(BinderInternal.getContextObject());
        return sServiceManager;
    }
```

采用了单例模式获取ServiceManager getIServiceManager()返回的是ServiceManagerProxy(简称SMP)对象

```
static public IServiceManager asInterface(IBinder obj)
    {
        if (obj == null) {
            return null;
        }
        IServiceManager in =
            (IServiceManager)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }

        return new ServiceManagerProxy(obj);
    }
```

`ServiceManagerProxy` 是`ServiceManagerNative`的内部类，实现IServiceManager接口

```
class ServiceManagerProxy implements IServiceManager {
    public ServiceManagerProxy(IBinder remote) {
        mRemote = remote;
    }

    public IBinder asBinder() {
        return mRemote;
    }

    public IBinder getService(String name) throws RemoteException {
       ...
    }

    public IBinder checkService(String name) throws RemoteException {
        ...
    }

    public void addService(String name, IBinder service, boolean allowIsolated, int dumpPriority)
            throws RemoteException {
       ...
    }

    public String[] listServices(int dumpPriority) throws RemoteException {
       ...
    }

    public void setPermissionController(IPermissionController controller)
            throws RemoteException {
       ...
    }

    private IBinder mRemote;
}

```

### getContextObject()

```
/**
     * Return the global "context object" of the system.  This is usually
     * an implementation of IServiceManager, which you can use to find
     * other services.
     */
    public static final native IBinder getContextObject();
```

调用jni中方法`android_os_BinderInternal_getContextObject`返回IBinder对象。

```
static jobject android_os_BinderInternal_getContextObject(JNIEnv* env, jobject clazz)
{
    sp<IBinder> b = ProcessState::self()->getContextObject(NULL);
    return javaObjectForIBinder(env, b);
}
```

对于ProcessState::self()->getContextObject()，在[获取ServiceManager](http://gityuan.com/2015/11/08/binder-get-sm/)的第3节已详细解决，即`ProcessState::self()->getContextObject()`等价于 `new BpBinder(0)`;

#### javaObjectForIBinder

[-> android_util_binder.cpp]

```
jobject javaObjectForIBinder(JNIEnv* env, const sp<IBinder>& val) {
    if (val == NULL) return NULL;

    if (val->checkSubclass(&gBinderOffsets)) { //返回false
        jobject object = static_cast<JavaBBinder*>(val.get())->object();
        return object;
    }

    AutoMutex _l(mProxyLock);

    jobject object = (jobject)val->findObject(&gBinderProxyOffsets);
    if (object != NULL) { //第一次object为null
        jobject res = jniGetReferent(env, object);
        if (res != NULL) {
            return res;
        }
        android_atomic_dec(&gNumProxyRefs);
        val->detachObject(&gBinderProxyOffsets);
        env->DeleteGlobalRef(object);
    }

    //创建BinderProxy对象
    object = env->NewObject(gBinderProxyOffsets.mClass, gBinderProxyOffsets.mConstructor);
    if (object != NULL) {
        //BinderProxy.mObject成员变量记录BpBinder对象
        env->SetLongField(object, gBinderProxyOffsets.mObject, (jlong)val.get());
        val->incStrong((void*)javaObjectForIBinder);

        jobject refObject = env->NewGlobalRef(
                env->GetObjectField(object, gBinderProxyOffsets.mSelf));
        //将BinderProxy对象信息附加到BpBinder的成员变量mObjects中
        val->attachObject(&gBinderProxyOffsets, refObject,
                jnienv_to_javavm(env), proxy_cleanup);

        sp<DeathRecipientList> drl = new DeathRecipientList;
        drl->incStrong((void*)javaObjectForIBinder);
        //BinderProxy.mOrgue成员变量记录死亡通知对象
        env->SetLongField(object, gBinderProxyOffsets.mOrgue, reinterpret_cast<jlong>(drl.get()));

        android_atomic_inc(&gNumProxyRefs);
        incRefsCreated(env);
    }
    return object;
}
```

根据BpBinder(C++)生成BinderProxy(Java)对象. 主要工作是创建BinderProxy对象,并把BpBinder对象地址保存到BinderProxy.mObject成员变量. 到此，可知

```
ServiceManagerNative.asInterface(BinderInternal.getContextObject()) 
等价于
ServiceManagerNative.asInterface(new BinderProxy())
```

##  SMN.asInterface

[-> ServiceManagerNative.java]

```
 static public IServiceManager asInterface(IBinder obj)
{
    if (obj == null) { //obj为BpBinder
        return null;
    }
    //由于obj为BpBinder，该方法默认返回null
    IServiceManager in = (IServiceManager)obj.queryLocalInterface(descriptor);
    if (in != null) {
        return in;
    }
    return new ServiceManagerProxy(obj); //【见小节3.3.1】
}
```

由此，可知

```
ServiceManagerNative.asInterface(new BinderProxy()) 
等价于
new ServiceManagerProxy(new BinderProxy())
```

 为了方便，ServiceManagerProxy简称为SMP。

### ServiceManagerProxy初始化

[-> ServiceManagerNative.java ::ServiceManagerProxy]

```
class ServiceManagerProxy implements IServiceManager {
    public ServiceManagerProxy(IBinder remote) {
        mRemote = remote;
    }
}
```

mRemote为BinderProxy对象，该BinderProxy对象对应于BpBinder(0)，其作为binder代理端，指向native层大管家service Manager。

`ServiceManager.getIServiceManager`最终等价于`new ServiceManagerProxy(new BinderProxy())`,意味着【3.1】中的getIServiceManager().addService()，等价于SMP.addService().

framework层的ServiceManager的调用实际的工作确实交给SMP的成员变量BinderProxy；而BinderProxy通过jni方式，最终会调用BpBinder对象；可见上层binder架构的核心功能依赖native架构的服务来完成的。

## ServiceManagerProxy.addService

[-> ServiceManagerNative.java ::ServiceManagerProxy]

```
public void addService(String name, IBinder service, boolean allowIsolated) throws RemoteException {
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();
    data.writeInterfaceToken(IServiceManager.descriptor);
    data.writeString(name);
    //【见小节3.5】
    data.writeStrongBinder(service);
    data.writeInt(allowIsolated ? 1 : 0);
    //mRemote为BinderProxy【见小节3.7】
    mRemote.transact(ADD_SERVICE_TRANSACTION, data, reply, 0);
    reply.recycle();
    data.recycle();
}
```

## data.writeStrongBinder(service)Java

[-> Parcel.java]

```
public writeStrongBinder(IBinder val){
    //此处为Native调用【见3.5.1】
    nativewriteStrongBinder(mNativePtr, val);
}
```

### nativeWriteStrongBinder()JNI

```
private static native void nativeWriteStrongBinder(long nativePtr, IBinder val);
```

[-> android_os_Parcel.cpp]

```
static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object) {
    //将java层Parcel转换为native层Parcel
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        //【见3.5.2】
        const status_t err = parcel->writeStrongBinder(ibinderForJavaObject(env, object));
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}
```

#### ibinderForJavaObject

[-> android_util_Binder.cpp]

```

sp<IBinder> ibinderForJavaObject(JNIEnv* env, jobject obj)
{
    if (obj == NULL) return NULL;

    //Java层的Binder对象
    if (env->IsInstanceOf(obj, gBinderOffsets.mClass)) {
        JavaBBinderHolder* jbh = (JavaBBinderHolder*)
            env->GetLongField(obj, gBinderOffsets.mObject);
        return jbh != NULL ? jbh->get(env, obj) : NULL; //【见3.5.3】
    }
    //Java层的BinderProxy对象
    if (env->IsInstanceOf(obj, gBinderProxyOffsets.mClass)) {
        return (IBinder*)env->GetLongField(obj, gBinderProxyOffsets.mObject);
    }
    return NULL;
}
```

根据Binde(Java)生成JavaBBinderHolder(C++)对象. 主要工作是创建JavaBBinderHolder对象,并把JavaBBinderHolder对象地址保存到Binder.mObject成员变量.

#### JavaBBinderHolder.get()

[-> android_util_Binder.cpp]

```
sp<JavaBBinder> get(JNIEnv* env, jobject obj)
{
    AutoMutex _l(mLock);
    sp<JavaBBinder> b = mBinder.promote();
    if (b == NULL) {
        //首次进来，创建JavaBBinder对象【见3.5.4】
        b = new JavaBBinder(env, obj);
        mBinder = b;
    }
    return b;
}
```

JavaBBinderHolder有一个成员变量mBinder，保存当前创建的JavaBBinder对象，这是一个wp类型的，可能会被垃圾回收器给回收，所以每次使用前，都需要先判断是否存在。

#### JavaBBinder初始化

==> [-> android_util_Binder.cpp]

```
JavaBBinder(JNIEnv* env, jobject object)
    : mVM(jnienv_to_javavm(env)), mObject(env->NewGlobalRef(object))
{
    android_atomic_inc(&gNumLocalRefs);
    incRefsCreated(env);
}
```

创建JavaBBinder，该对象继承于BBinder对象。

```
data.writeStrongBinder(service)
最终等价于`parcel->
writeStrongBinder(new JavaBBinder(env, obj))`;
```

### writeStrongBinder()C++

[-> parcel.cpp]

```
status_t Parcel::writeStrongBinder(const sp<IBinder>& val)
{
    return flatten_binder(ProcessState::self(), val, this);
}
```

#### flatten_binder

[-> parcel.cpp]

```

status_t flatten_binder(const sp<ProcessState>& /*proc*/,
    const sp<IBinder>& binder, Parcel* out)
{
    flat_binder_object obj;

    obj.flags = 0x7f | FLAT_BINDER_FLAG_ACCEPTS_FDS;
    if (binder != NULL) {
        IBinder *local = binder->localBinder();
        if (!local) {
            BpBinder *proxy = binder->remoteBinder();
            const int32_t handle = proxy ? proxy->handle() : 0;
            obj.type = BINDER_TYPE_HANDLE; //远程Binder
            obj.binder = 0;
            obj.handle = handle;
            obj.cookie = 0;
        } else {
            obj.type = BINDER_TYPE_BINDER; //本地Binder，进入该分支
            obj.binder = reinterpret_cast<uintptr_t>(local->getWeakRefs());
            obj.cookie = reinterpret_cast<uintptr_t>(local);
        }
    } else {
        obj.type = BINDER_TYPE_BINDER;  //本地Binder
        obj.binder = 0;
        obj.cookie = 0;
    }
    //【见小节3.6.2】
    return finish_flatten_binder(binder, obj, out);
}
```

将Binder对象扁平化，转换成flat_binder_object对象。

- 对于Binder实体，则cookie记录Binder实体的指针；
- 对于Binder代理，则用handle记录Binder代理的句柄；

关于localBinder，代码见Binder.cpp。

```
BBinder* BBinder::localBinder()
{
    return this;
}

BBinder* IBinder::localBinder()
{
    return NULL;
}
```

#### finish_flatten_binder

```
inline static status_t finish_flatten_binder(
    const sp<IBinder>& , const flat_binder_object& flat, Parcel* out)
{
    return out->writeObject(flat, false);
}
```

会带Java代码addService过程，则接下来进入transact

## BinderProxy.transact

[-> Binder.java ::BinderProxy]

```
public boolean transact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
    //用于检测Parcel大小是否大于800k
    Binder.checkParcel(this, code, data, "Unreasonably large binder buffer");
    return transactNative(code, data, reply, flags); //【见3.8】
}

```

回到ServiceManagerProxy.addService，其成员变量mRemote是BinderProxy。transactNative经过jni调用，进入下面的方法

### android_os_BinderProxy_transact

[-> android_util_Binder.cpp]

```
static jboolean android_os_BinderProxy_transact(JNIEnv* env, jobject obj,
    jint code, jobject dataObj, jobject replyObj, jint flags)
{
    ...
    //java Parcel转为native Parcel
    Parcel* data = parcelForJavaObject(env, dataObj);
    Parcel* reply = parcelForJavaObject(env, replyObj);
    ...

    //gBinderProxyOffsets.mObject中保存的是new BpBinder(0)对象
    IBinder* target = (IBinder*)
        env->GetLongField(obj, gBinderProxyOffsets.mObject);
    ...

    //此处便是BpBinder::transact(), 经过native层，进入Binder驱动程序
    status_t err = target->transact(code, *data, reply, flags);
    ...
    return JNI_FALSE;
}
```

Java层的BinderProxy.transact()最终交由Native层的BpBinder::transact()完成。Native Binder的[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)中有详细说明BpBinder执行过程。另外，该方法可抛出RemoteException。

## 小结

addService的核心过程：

```
public void addService(String name, IBinder service, boolean allowIsolated) throws RemoteException {
    ...
    Parcel data = Parcel.obtain(); //此处还需要将java层的Parcel转为Native层的Parcel
    data->writeStrongBinder(new JavaBBinder(env, obj));
    BpBinder::transact(ADD_SERVICE_TRANSACTION, *data, reply, 0); //与Binder驱动交互
    ...
}
```

注册服务过程就是通过BpBinder来发送`ADD_SERVICE_TRANSACTION`命令，与实现与binder驱动进行数据交互。

# 获取服务

## SM.getService

```
public static IBinder getService(String name) {
    try {
        IBinder service = sCache.get(name); //先从缓存中查看HashMap
        if (service != null) {
            return service;
        } else {```
            return getIServiceManager().getService(name); 【见4.2】
        }
    } catch (RemoteException e) {
        Log.e(TAG, "error in getService", e);
    }
    return null;
}
```

关于getIServiceManager()，在前面已经讲述了，等价于new ServiceManagerProxy(new BinderProxy())。 其中sCache = new HashMap<String, IBinder>()以hashmap格式缓存已组成的名称。请求获取服务过程中，先从缓存中查询是否存在，如果缓存中不存在的话，再通过binder交互来查询相应的服务。

## ServiceManagerProxy.getService()

```
class ServiceManagerProxy implements IServiceManager {
    public IBinder getService(String name) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeString(name);
        //mRemote为BinderProxy 【见4.3】
        mRemote.transact(GET_SERVICE_TRANSACTION, data, reply, 0);
        //从reply里面解析出获取的IBinder对象【见4.8】
        IBinder binder = reply.readStrongBinder();
        reply.recycle();
        data.recycle();
        return binder;
    }
}
```

## BinderProxy.transact

[-> Binder.java]

```
final class BinderProxy implements IBinder {
    public boolean transact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        Binder.checkParcel(this, code, data, "Unreasonably large binder buffer");
        return transactNative(code, data, reply, flags);
    }
}
```

## android_os_BinderProxy_transact

[-> android_util_Binder.cpp]

```
static jboolean android_os_BinderProxy_transact(JNIEnv* env, jobject obj,
    jint code, jobject dataObj, jobject replyObj, jint flags)
{
    ...
    //java Parcel转为native Parcel
    Parcel* data = parcelForJavaObject(env, dataObj);
    Parcel* reply = parcelForJavaObject(env, replyObj);
    ...

    //gBinderProxyOffsets.mObject中保存的是new BpBinder(0)对象
    IBinder* target = (IBinder*)
        env->GetLongField(obj, gBinderProxyOffsets.mObject);
    ...

    //此处便是BpBinder::transact(), 经过native层[见小节4.5]
    status_t err = target->transact(code, *data, reply, flags);
    ...
    return JNI_FALSE;
}
```

## BpBinder.transact

```
status_t BpBinder::transact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    if (mAlive) {
        // [见小节4.6]
        status_t status = IPCThreadState::self()->transact(
            mHandle, code, data, reply, flags);
        if (status == DEAD_OBJECT) mAlive = 0;
        return status;
    }

    return DEAD_OBJECT;
```

## IPC.transact

[-> IPCThreadState.cpp]

```
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel& data,
                                  Parcel* reply, uint32_t flags)
{
    status_t err = data.errorCheck(); //数据错误检查
    flags |= TF_ACCEPT_FDS;
    ....
    if (err == NO_ERROR) {
         // 传输数据
        err = writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);
    }
    ...

    // 默认情况下,都是采用非oneway的方式, 也就是需要等待服务端的返回结果
    if ((flags & TF_ONE_WAY) == 0) {
        if (reply) {
            //等待回应事件
            err = waitForResponse(reply);
        }else {
            Parcel fakeReply;
            err = waitForResponse(&fakeReply);
        }
    } else {
        err = waitForResponse(NULL, NULL);
    }
    return err;
}
```

##  IPC.waitForResponse

```
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel& data,
                                  Parcel* reply, uint32_t flags)
{
    status_t err = data.errorCheck(); //数据错误检查
    flags |= TF_ACCEPT_FDS;
    ....
    if (err == NO_ERROR) {
         // 传输数据
        err = writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);
    }
    ...

    // 默认情况下,都是采用非oneway的方式, 也就是需要等待服务端的返回结果
    if ((flags & TF_ONE_WAY) == 0) {
        if (reply) {
            //等待回应事件
            err = waitForResponse(reply);
        }else {
            Parcel fakeReply;
            err = waitForResponse(&fakeReply);
        }
    } else {
        err = waitForResponse(NULL, NULL);
    }
    return err;
}
```

##  IPC.waitForResponse

```
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
    int32_t cmd;
    int32_t err;
    while (1) {
        if ((err=talkWithDriver()) < NO_ERROR) break;
        ...
        cmd = mIn.readInt32();
        switch (cmd) {
          case BR_REPLY:
          {
            binder_transaction_data tr;
            err = mIn.read(&tr, sizeof(tr));
            if (reply) {
                if ((tr.flags & TF_STATUS_CODE) == 0) {
                    //当reply对象回收时，则会调用freeBuffer来回收内存
                    reply->ipcSetDataReference(
                        reinterpret_cast<const uint8_t*>(tr.data.ptr.buffer),
                        tr.data_size,
                        reinterpret_cast<const binder_size_t*>(tr.data.ptr.offsets),
                        tr.offsets_size/sizeof(binder_size_t),
                        freeBuffer, this);
                } else {
                    ...
                }
            }
          }
          case :...
        }
    }
    ...
    return err;
}
```

那么这个reply是哪来的呢，在文章[Binder系列3—启动ServiceManager](http://gityuan.com/2015/11/07/binder-start-sm/)

### binder_send_reply

[-> servicemanager/binder.c]

```
void binder_send_reply(struct binder_state *bs, struct binder_io *reply, binder_uintptr_t buffer_to_free, int status) {
    struct {
        uint32_t cmd_free;
        binder_uintptr_t buffer;
        uint32_t cmd_reply;
        struct binder_transaction_data txn;
    } __attribute__((packed)) data;

    data.cmd_free = BC_FREE_BUFFER; //free buffer命令
    data.buffer = buffer_to_free;
    data.cmd_reply = BC_REPLY; // reply命令
    data.txn.target.ptr = 0;
    data.txn.cookie = 0;
    data.txn.code = 0;
    if (status) {
        ...
    } else {=

        data.txn.flags = 0;
        data.txn.data_size = reply->data - reply->data0;
        data.txn.offsets_size = ((char*) reply->offs) - ((char*) reply->offs0);
        data.txn.data.ptr.buffer = (uintptr_t)reply->data0;
        data.txn.data.ptr.offsets = (uintptr_t)reply->offs0;
    }
    //向Binder驱动通信
    binder_write(bs, &data, sizeof(data));
}
```

binder_write将BC_FREE_BUFFER和BC_REPLY命令协议发送给驱动，进入驱动。binder_ioctl -> binder_ioctl_write_read -> binder_thread_write，由于是BC_REPLY命令协议，则进入binder_transaction， 该方法会向请求服务的线程Todo队列插入事务。

接下来，请求服务的进程在执行talkWithDriver的过程执行到binder_thread_read()，处理Todo队列的事务。

## readStrongBinder

[-> Parcel.java]

readStrongBinder的过程基本是writeStrongBinder逆过程。

```
static jobject android_os_Parcel_readStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr) {
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        //【见小节4.8.1】
        return javaObjectForIBinder(env, parcel->readStrongBinder());
    }
    return NULL;
}
```

javaObjectForIBinder 将native层BpBinder对象转换为Java层BinderProxy对象。

### readStrongBinder(C++)

[-> Parcel.cpp]

```
sp<IBinder> Parcel::readStrongBinder() const
{
    sp<IBinder> val;
    //【见小节4.8.2】
    unflatten_binder(ProcessState::self(), *this, &val);
    return val;
}
```

### unflatten_binder

```
status_t unflatten_binder(const sp<ProcessState>& proc,
    const Parcel& in, sp<IBinder>* out)
{
    const flat_binder_object* flat = in.readObject(false);
    if (flat) {
        switch (flat->type) {
            case BINDER_TYPE_BINDER:
                *out = reinterpret_cast<IBinder*>(flat->cookie);
                return finish_unflatten_binder(NULL, *flat, in);
            case BINDER_TYPE_HANDLE:
                //进入该分支【见4.8.3】
                *out = proc->getStrongProxyForHandle(flat->handle);
                //创建BpBinder对象
                return finish_unflatten_binder(
                    static_cast<BpBinder*>(out->get()), *flat, in);
        }
    }
    return BAD_TYPE;
}
```

### getStrongProxyForHandle

```
sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
{
    sp<IBinder> result;

    AutoMutex _l(mLock);
    //查找handle对应的资源项
    handle_entry* e = lookupHandleLocked(handle);

    if (e != NULL) {
        IBinder* b = e->binder;
        if (b == NULL || !e->refs->attemptIncWeak(this)) {
            ...
            //当handle值所对应的IBinder不存在或弱引用无效时，则创建BpBinder对象
            b = new BpBinder(handle);
            e->binder = b;
            if (b) e->refs = b->getWeakRefs();
            result = b;
        } else {
            result.force_set(b);
            e->refs->decWeak(this);
        }
    }
    return result;
}
```

经过该方法，最终创建了指向Binder服务端的BpBinder代理对象。回到[小节4.8] 经过javaObjectForIBinder将native层BpBinder对象转换为Java层BinderProxy对象。 也就是说通过getService()最终获取了指向目标Binder服务端的代理对象BinderProxy。

## 小结

getService的核心过程：

```
public static IBinder getService(String name) {
    ...
    Parcel reply = Parcel.obtain(); //此处还需要将java层的Parcel转为Native层的Parcel
    BpBinder::transact(GET_SERVICE_TRANSACTION, *data, reply, 0);  //与Binder驱动交互
    IBinder binder = javaObjectForIBinder(env, new BpBinder(handle));
    ...
}
```

javaObjectForIBinder作用是创建BinderProxy对象，并将BpBinder对象的地址保存到BinderProxy对象的mObjects中。 获取服务过程就是通过BpBinder来发送`GET_SERVICE_TRANSACTION`命令，与实现与binder驱动进行数据交互。

# 实例

以IWindowManager为例

```
public interface IWindowManager extends android.os.IInterface {

    public static abstract class Stub extends android.os.Binder implements android.view.IWindowManager {
        private static final java.lang.String DESCRIPTOR = "android.view.IWindowManager";

        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        public static android.view.IWindowManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof android.view.IWindowManager))) {
                return ((android.view.IWindowManager) iin);
            }
            return new android.view.IWindowManager.Stub.Proxy(obj);
        }

        public android.os.IBinder asBinder() {
            return this;
        }

        private static class Proxy implements android.view.IWindowManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            public android.os.IBinder asBinder() {
                return mRemote;
            }
        }
        ...
    }
}
```

## Binder

```
public class Binder implements IBinder {
    public void attachInterface(IInterface owner, String descriptor) {
        mOwner = owner;
        mDescriptor = descriptor;
    }

    public IInterface queryLocalInterface(String descriptor) {
        if (mDescriptor.equals(descriptor)) {
            return mOwner;
        }
        return null;
    }
}
```

## BinderProxy

```
final class BinderProxy implements IBinder {
    public IInterface queryLocalInterface(String descriptor) {
        return null;
    }
}
```



