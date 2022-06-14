---
title: How to write a file-watcher and read changes
slug: write-a-file-watcher-and-read-the-changes
date_published: 2022-02-20T09:34:47.000Z
date_updated: 2022-02-20T09:34:47.000Z
---

While working on an on-premise compute grid, there was a need to monitor the status of various jobs running on worker nodes. There is a scheduler which assign tasks to worker nodes and maintains the status of each job. As a job moves through various stages of its life cycle, scheduler captures these events and maintains in a log file. For a user, these changes in the status provide the progress that job is making or any error it encounters.

Idea was to deploy a file watcher application on scheduler node which can monitor a directory . This is the directory where all the log files are present. As and when these log files are updated, file-watcher can detect that change and take an action , which could be sending this event to a message broker for others to consume.

The file watcher is written in python 3.X. We are going to use python package [watchdog](https://pypi.org/project/watchdog/) which provides a way to monitor directory and inform whenever a change occurs.* It only tell you that something has changed in a given file and NOT what has changed*. For that, we need to read the file. Since there are hundreds-of-thousands jobs are running and that many files may be be available, what we wanted was, to  read a file, get the latest update and close the file so that we are not keeping so many file handlers in the memory. Since we are opening and reading this file every time this is updated, we only need to read from previous point on wards and not the whole file every time.

When dealing with files in python, file handler exposes a function ,**tell(), ** which return the current stream position. Initially this starts with zero. Every time, we finished reading file, we store this position against file name in a dict. When start reading a file, get this position, seek that position and read from there. With this, we don't need to read the whole file and can extract only what has changed since last read.

           
    def get_file_contents(self, file, position):
            """
            reads the contents of the file from last position and
            return contents with current position
            """
            with open(file, 'r') as f:
                f.seek(position)
                return f.read(), f.tell()

This is an approach to extract the delta from those log files and depending on the latency and performance , may require some tweaks.

code - [https://github.com/kapilgarg/file-watcher](https://github.com/kapilgarg/file-watcher)
