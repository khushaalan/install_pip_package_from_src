### Installing `quickfix` on Apple Silicon (M1/M2) Macs

In this example, we will take `quickfix` as an example.  
If you try to run:

```bash
pip install quickfix
```

on an Apple Silicon Mac, you will encounter a few warnings or build issues.  
The simple workaround is to edit the source code and rebuild the package.

We will be referring to this tutorial:  
https://medium.com/@a.anekpattanakij/journey-to-install-the-quickfix-for-python-on-mac-m1-arm64-1f57f2717d9

---

#### Steps:

1. Go to the PyPI page for quickfix:  
   https://pypi.org/project/quickfix/

2. Download the source archive `quickfix-1.15.1.tar.gz`.

3. Extract the archive and locate the file:  
   `src/C++/AtomicCount.h`

4. Replace the contents of `AtomicCount.h` with the following:

```cpp
/* -*- C++ -*- */

/****************************************************************************
** Copyright (c) 2001-2014
**
** This file is part of the QuickFIX FIX Engine
**
** This file may be distributed under the terms of the quickfixengine.org
** license as defined by quickfixengine.org and appearing in the file
** LICENSE included in the packaging of this file.
**
** This file is provided AS IS with NO WARRANTY OF ANY KIND, INCLUDING THE
** WARRANTY OF DESIGN, MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.
**
** See http://www.quickfixengine.org/LICENSE for licensing information.
**
** Contact ask@quickfixengine.org if any conditions of this licensing are
** not clear to you.
**
****************************************************************************/

// #ifndef ATOMIC_COUNT
// #define ATOMIC_COUNT

// #include "Utility.h"

// #if defined(__SUNPRO_CC) ||  defined(__TOS_AIX__)
// #include "Mutex.h"
// #endif

// namespace FIX
// {
//   /// Atomic count class - consider using interlocked functions

// #ifdef ENABLE_BOOST_ATOMIC_COUNT

// #include <boost/smart_ptr/detail/atomic_count.hpp>
// typedef boost::detail::atomic_count atomic_count;

// #elif _MSC_VER 

//   //atomic counter based on interlocked functions for Win32
//   class atomic_count
//   {
//   public:
//     explicit atomic_count( long v ): m_counter( v )
//     {
//     }

//     long operator++()
//     {
//       return ::InterlockedIncrement( &m_counter );
//     }

//     long operator--()
//     {
//       return ::InterlockedDecrement( &m_counter );
//     }

//     operator long() const
//     {
//       return ::InterlockedExchangeAdd(const_cast<long volatile *>( &m_counter ), 0 );
//     }

//   private:

//     atomic_count( atomic_count const & );
//     atomic_count & operator=( atomic_count const & );

//     long volatile m_counter;
//   };

// #elif defined(__SUNPRO_CC) ||  defined(__TOS_AIX__)

// // general purpose atomic counter using mutexes
// class atomic_count
// {
// public:
//   explicit atomic_count( long v ): m_counter( v )
//   {
//   }

//   long operator++()
//   {
//     Locker _lock(m_mutex);
//     return ++m_counter;
//   }

//   long operator--()
//   {
//     Locker _lock(m_mutex);
//     return --m_counter;
//   }

//   operator long() const
//   {
//     return static_cast<long const volatile &>( m_counter );
//   }

// private:

//   atomic_count( atomic_count const & );
//   atomic_count & operator=( atomic_count const & );

//   Mutex m_mutex;
//   long m_counter;
// };

// #else

//   //
//   //  boost/detail/atomic_count_gcc_x86.hpp
//   //
//   //  atomic_count for g++ on 486+/AMD64
//   //
//   //  Copyright 2007 Peter Dimov
//   //
//   //  Distributed under the Boost Software License, Version 1.0. (See
//   //  accompanying file LICENSE_1_0.txt or copy at
//   //  http://www.boost.org/LICENSE_1_0.txt)
//   //

//   class atomic_count
//   {
//   public:

//     explicit atomic_count( long v ) : value_(static_cast<int>(v)) {}

//     long operator++()
//     {
//       return atomic_exchange_and_add( &value_, 1 ) + 1;
//     }

//     long operator--()
//     {
//       return atomic_exchange_and_add( &value_, -1 ) - 1;
//     }

//     operator long() const
//     {
//       return atomic_exchange_and_add( &value_, 0 );
//     }

//   private:

//     atomic_count( atomic_count const & );
//     atomic_count & operator=( atomic_count const & );

//     mutable int value_;

//   private:

//     static int atomic_exchange_and_add(int * pw, int dv)
//     {
//       // int r = *pw;
//       // *pw += dv;
//       // return r;

//       int r;

//       __asm__ __volatile__
//         (
//           "lock\n\t"
//           "xadd %1, %0":
//           "+m"(*pw), "=r"(r) : // outputs (%0, %1)
//           "1"(dv) : // inputs (%2 == %1)
//           "memory", "cc" // clobbers
//         );

//       return r;
//     }
//   };

// #endif

// }

// #endif




#ifndef ATOMIC_COUNT
#define ATOMIC_COUNT

#include "Utility.h"

#if defined(__SUNPRO_CC) ||  defined(__TOS_AIX__)
#include "Mutex.h"
#endif

namespace FIX
{
  /// Atomic count class - consider using interlocked functions

#ifdef ENABLE_BOOST_ATOMIC_COUNT

#include <boost/smart_ptr/detail/atomic_count.hpp>
typedef boost::detail::atomic_count atomic_count;

#elif _MSC_VER 

  //atomic counter based on interlocked functions for Win32
  class atomic_count
  {
  public:
    explicit atomic_count( long v ): m_counter( v )
    {
    }

    long operator++()
    {
      return ::InterlockedIncrement( &m_counter );
    }

    long operator--()
    {
      return ::InterlockedDecrement( &m_counter );
    }

    operator long() const
    {
      return ::InterlockedExchangeAdd(const_cast<long volatile *>( &m_counter ), 0 );
    }

  private:

    atomic_count( atomic_count const & );
    atomic_count & operator=( atomic_count const & );

    long volatile m_counter;
  };

#elif defined(__SUNPRO_CC) ||  defined(__TOS_AIX__)

// general purpose atomic counter using mutexes
class atomic_count
{
public:
  explicit atomic_count( long v ): m_counter( v )
  {
  }

  long operator++()
  {
    Locker _lock(m_mutex);
    return ++m_counter;
  }

  long operator--()
  {
    Locker _lock(m_mutex);
    return --m_counter;
  }

  operator long() const
  {
    return static_cast<long const volatile &>( m_counter );
  }

private:

  atomic_count( atomic_count const & );
  atomic_count & operator=( atomic_count const & );

  Mutex m_mutex;
  long m_counter;
};

#else


  class atomic_count
  {
  public:

    explicit atomic_count( long v ) : value_(static_cast<int>(v)) {}

    long operator++()
    {
      return atomic_exchange_and_add( &value_, 1 ) + 1;
    }

    long operator--()
    {
      return atomic_exchange_and_add( &value_, -1 ) - 1;
    }

    operator long() const
    {
      return atomic_exchange_and_add( &value_, 0 );
    }

  private:

    atomic_count( atomic_count const & );
    atomic_count & operator=( atomic_count const & );

    mutable int value_;

  private:

    static int atomic_exchange_and_add(int * pw, int dv)
    {
       int r = *pw;
       *pw += dv;
       return r;

    }
  };

#endif

}

#endif
```

5. Re-compress the modified source folder:

```bash
tar -czvf test1.tar.gz quickfix-1.15.1
```

6. Finally, install the rebuilt package locally:

```bash
pip install test1.tar.gz
```

![image](https://github.com/user-attachments/assets/8f5e03e0-10a0-436b-9e4e-cfb6b9f75ba9)

