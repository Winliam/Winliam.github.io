---
title: gdb调试插件
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-09-01 02:18:19
tags:
  - gdb
  - gdb插件
  - debug
  - Eigen
  - OpenCV
---
## 前言
使用gdb进行调试时，最重要的功能就是看变量的当前值。

但是，当这个需求遇到OpenCV中的Mat或者Eigen中的Matrix时，就变得不太好搞。因为这些矩阵很大，又往往被封装的比较复杂，所以直接打印出来对调试而言非常不友好。

因此，便可以借助python对于gdb的支持，来搞一些插件，来优化打印结果。

这里先介绍一下gdb插件加载机制的原理，然后介绍两个目前找到的比较好的插件，分别针对Eigen Matrix和OpenCV Mat打印。

## gdb插件加载原理
mark

## cv_show插件
该插件可以在gdb调试模式下将cv::Mat数据绘制出来，并且支持放缩，保存，以及像素值的鼠标悬浮显示等功能，还是比较好用的。

具体用法也很简单，首先将以下代码另存为一个python文件，然后在gdb的命令行中source该文件，然后命令行中输入`cv_imshow img_name`即可图像显示出来。
```python
# Copyright (c) 2012, Renato Florentino Garcia <fgarcia.renato@gmail.com>
#                     Stefano Pellegrini <stefpell@ee.ethz.ch>
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
#     * Neither the name of the authors nor the
#       names of its contributors may be used to endorse or promote products
#       derived from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE AUTHORS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

from builtins import str as text
from builtins import range

import gdb
import matplotlib
matplotlib.use('TKAgg')
import matplotlib.pyplot as pl

import numpy as np
import struct


def chunker(seq, size):
    return (seq[pos:pos + size] for pos in range(0, len(seq), size))

class cv_imshow(gdb.Command):
    """Diplays the content of an opencv image"""

    def __init__(self):
        super(cv_imshow, self).__init__('cv_imshow',
                                        gdb.COMMAND_SUPPORT,
                                        gdb.COMPLETE_SYMBOL)

    def invoke (self, arg, from_tty):
        # Access the variable from gdb.
        args = gdb.string_to_argv(arg)
        val = gdb.parse_and_eval(args[0])
        if str(val.type.strip_typedefs()) == 'IplImage *':
            img_info = self.get_iplimage_info(val)
        else:
            img_info = self.get_cvmat_info(val)

        if img_info: self.show_image(*img_info)

        self.dont_repeat()

    @staticmethod
    def get_cvmat_info(val):
        flags = val['flags']
        depth = flags & 7
        channels = 1 + (flags >> 3) & 63;
        if depth == 0:
            cv_type_name = 'CV_8U'
            data_symbol = 'B'
        elif depth == 1:
            cv_type_name = 'CV_8S'
            data_symbol = 'b'
        elif depth == 2:
            cv_type_name = 'CV_16U'
            data_symbol = 'H'
        elif depth == 3:
            cv_type_name = 'CV_16S'
            data_symbol = 'h'
        elif depth == 4:
            cv_type_name = 'CV_32S'
            data_symbol = 'i'
        elif depth == 5:
            cv_type_name = 'CV_32F'
            data_symbol = 'f'
        elif depth == 6:
            cv_type_name = 'CV_64F'
            data_symbol = 'd'
        else:
            gdb.write('Unsupported cv::Mat depth\n', gdb.STDERR)
            return

        rows = val['rows']
        cols = val['cols']

        line_step = val['step']['p'][0]

        gdb.write(cv_type_name + ' with ' + str(channels) + ' channels, ' +
                  str(rows) + ' rows and ' +  str(cols) +' cols\n')

        data_address = text(val['data']).encode('utf-8').split()[0]
        data_address = int(data_address, 16)

        return (cols, rows, channels, line_step, data_address, data_symbol)

    @staticmethod
    def get_iplimage_info(val):
        depth = val['depth']
        channels = val['nChannels']
        if depth == 0x8:
            cv_type_name = 'IPL_DEPTH_8U'
            data_symbol = 'B'
            elem_size = 1
        elif depth == -0x7FFFFFF8:
            cv_type_name = 'IPL_DEPTH_8S'
            data_symbol = 'b'
            elem_size = 1
        elif depth == 0x10:
            cv_type_name = 'IPL_DEPTH_16U'
            data_symbol = 'H'
            elem_size = 2
        elif depth == -0x7FFFFFF0:
            cv_type_name = 'IPL_DEPTH_16S'
            data_symbol = 'h'
            elem_size = 2
        elif depth == -0x7FFFFFE0:
            cv_type_name = 'IPL_DEPTH_32S'
            data_symbol = 'i'
            elem_size = 4
        elif depth == 0x20:
            cv_type_name = 'IPL_DEPTH_32F'
            data_symbol = 'f'
            elem_size = 4
        elif depth == 0x40:
            cv_type_name = 'IPL_DEPTH_64F'
            data_symbol = 'd'
            elem_size = 8
        else:
            gdb.write('Unsupported IplImage depth\n', gdb.STDERR)
            return

        rows = val['height'] if str(val['roi']) == '0x0' else val['roi']['height']
        cols = val['width'] if str(val['roi']) == '0x0' else val['roi']['width']
        line_step = val['widthStep']

        gdb.write(cv_type_name + ' with ' + str(channels) + ' channels, ' +
                  str(rows) + ' rows and ' +  str(cols) +' cols\n')

        data_address = text(val['imageData']).encode('utf-8').split()[0]
        data_address = int(data_address, 16)
        if str(val['roi']) != '0x0':
            x_offset = int(val['roi']['xOffset'])
            y_offset = int(val['roi']['yOffset'])
            data_address += line_step * y_offset + x_offset * elem_size * channels

        return (cols, rows, channels, line_step, data_address, data_symbol)


    @staticmethod
    def show_image(width, height, n_channel, line_step, data_address, data_symbol):
        """ Copies the image data to a PIL image and shows it.

        Args:
            width: The image width, in pixels.
            height: The image height, in pixels.
            n_channel: The number of channels in image.
            line_step: The offset to change to pixel (i+1, j) being
                in pixel (i, j), in bytes.
            data_address: The address of image data in memory.
            data_symbol: Python struct module code to the image data type.
        """
        width = int(width)
        height = int(height)
        n_channel = int(n_channel)
        line_step = int(line_step)
        data_address = int(data_address)

        infe = gdb.inferiors()
        memory_data = infe[0].read_memory(data_address, line_step * height)

        # Calculate the memory padding to change to the next image line.
        # Either due to memory alignment or a ROI.
        if data_symbol in ('b', 'B'):
            elem_size = 1
        elif data_symbol in ('h', 'H'):
            elem_size = 2
        elif data_symbol in ('i', 'f'):
            elem_size = 4
        elif data_symbol == 'd':
            elem_size = 8
        padding = line_step - width * n_channel * elem_size

        # Format memory data to load into the image.
        image_data = []
        if n_channel == 1:
            mode = 'L'
            fmt = '%d%s%dx' % (width, data_symbol, padding)
            for line in chunker(memory_data, line_step):
                image_data.extend(struct.unpack(fmt, line))
        elif n_channel == 3:
            mode = 'RGB'
            fmt = '%d%s%dx' % (width * 3, data_symbol, padding)
            for line in chunker(memory_data, line_step):
                image_data.extend(struct.unpack(fmt, line))
        else:
            gdb.write('Only 1 or 3 channels supported\n', gdb.STDERR)
            return

        # Fit the opencv elemente data in the PIL element data
        if data_symbol == 'b':
            image_data = [i+128 for i in image_data]
        elif data_symbol == 'H':
            image_data = [i>>8 for i in image_data]
        elif data_symbol == 'h':
            image_data = [(i+32768)>>8 for i in image_data]
        elif data_symbol == 'i':
            image_data = [(i+2147483648)>>24 for i in image_data]
        elif data_symbol in ('f','d'):
            # A float image is discretized in 256 bins for display.
            max_image_data = max(image_data)
            min_image_data = min(image_data)
            img_range = max_image_data - min_image_data
            if img_range > 0:
                image_data = [int(255 * (i - min_image_data) / img_range) \
                              for i in image_data]
            else:
                image_data = [0 for i in image_data]


        if n_channel == 3:
            # OpenCV stores the channels in BGR mode. Convert to RGB while packing.
            image_data = list(zip(*[image_data[i::3] for i in [2, 1, 0]]))

        img = None
        if mode == 'L':
            img = np.reshape(image_data, (height, width)).astype(np.uint8)
        elif mode == 'RGB':
            img = np.reshape(image_data, (height, width, 3)).astype(np.uint8)

        fig = pl.figure()
        b = fig.add_subplot(111)
        if n_channel == 1:
            b.imshow(img, cmap = pl.cm.Greys_r, interpolation='nearest')
        elif n_channel == 3:
            b.imshow(img, interpolation='nearest')

        def format_coord(x, y):
            col = int(x+0.5)
            row = int(y+0.5)
            if col>=0 and col<width and row>=0 and row<height:
                if n_channel == 1:
                    z = img[row,col]
                    return '(%d, %d), [%1.2f]'%(col, row, z)
                elif n_channel == 3:
                    z0 = img[row,col,0]
                    z1 = img[row,col,1]
                    z2 = img[row,col,2]
                    return '(%d, %d), [%1.2f, %1.2f, %1.2f]'%(col, row, z0, z1, z2)
            else:
                return 'x=%d, y=%d'%(col, row)

        b.format_coord = format_coord
        pl.show()

cv_imshow()
```

## Eigen printer
这是Eigen的官方插件，用于Matrix print。

使用方法为:
1. 在本地新建一个目录，将以下代码保存到该目录下，命名为printer.py
2. 在该目录下另外新建一个空的__init__.py文件
3. 在用户目录下（~下）新建一个.gdbinit文件，文件内容如以下代码头部的相关注释所示，其中的路径要改成上述目录的绝对地址
4. 此时，进入gdb调试模式后正常使用print命令即可
5. 较大的矩阵，在打印时可能会被默认截断，用`set print elements unlimited`取消默认省略机制即可
```python
# -*- coding: utf-8 -*-
# This file is part of Eigen, a lightweight C++ template library
# for linear algebra.
#
# Copyright (C) 2009 Benjamin Schindler <bschindler@inf.ethz.ch>
#
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

# Pretty printers for Eigen::Matrix
# This is still pretty basic as the python extension to gdb is still pretty basic. 
# It cannot handle complex eigen types and it doesn't support many of the other eigen types
# This code supports fixed size as well as dynamic size matrices

# To use it:
#
# * Create a directory and put the file as well as an empty __init__.py in 
#   that directory.
# * Create a ~/.gdbinit file, that contains the following:
#      python
#      import sys
#      sys.path.insert(0, '/path/to/eigen/printer/directory')
#      from printers import register_eigen_printers
#      register_eigen_printers (None)
#      end

import gdb
import re
import itertools
from bisect import bisect_left

# Basic row/column iteration code for use with Sparse and Dense matrices
class _MatrixEntryIterator(object):
	
	def __init__ (self, rows, cols, rowMajor):
		self.rows = rows
		self.cols = cols
		self.currentRow = 0
		self.currentCol = 0
		self.rowMajor = rowMajor

	def __iter__ (self):
		return self

	def next(self):
                return self.__next__()  # Python 2.x compatibility

	def __next__(self):
		row = self.currentRow
		col = self.currentCol
		if self.rowMajor == 0:
			if self.currentCol >= self.cols:
				raise StopIteration
				
			self.currentRow = self.currentRow + 1
			if self.currentRow >= self.rows:
				self.currentRow = 0
				self.currentCol = self.currentCol + 1
		else:
			if self.currentRow >= self.rows:
				raise StopIteration
				
			self.currentCol = self.currentCol + 1
			if self.currentCol >= self.cols:
				self.currentCol = 0
				self.currentRow = self.currentRow + 1

		return (row, col)

class EigenMatrixPrinter:
	"Print Eigen Matrix or Array of some kind"

	def __init__(self, variety, val):
		"Extract all the necessary information"
		
		# Save the variety (presumably "Matrix" or "Array") for later usage
		self.variety = variety
		
		# The gdb extension does not support value template arguments - need to extract them by hand
		type = val.type
		if type.code == gdb.TYPE_CODE_REF:
			type = type.target()
		self.type = type.unqualified().strip_typedefs()
		tag = self.type.tag
		regex = re.compile('\<.*\>')
		m = regex.findall(tag)[0][1:-1]
		template_params = m.split(',')
		template_params = [x.replace(" ", "") for x in template_params]
		
		if template_params[1] == '-0x00000000000000001' or template_params[1] == '-0x000000001' or template_params[1] == '-1':
			self.rows = val['m_storage']['m_rows']
		else:
			self.rows = int(template_params[1])
		
		if template_params[2] == '-0x00000000000000001' or template_params[2] == '-0x000000001' or template_params[2] == '-1':
			self.cols = val['m_storage']['m_cols']
		else:
			self.cols = int(template_params[2])
		
		self.options = 0 # default value
		if len(template_params) > 3:
			self.options = template_params[3];
		
		self.rowMajor = (int(self.options) & 0x1)
		
		self.innerType = self.type.template_argument(0)
		
		self.val = val
		
		# Fixed size matrices have a struct as their storage, so we need to walk through this
		self.data = self.val['m_storage']['m_data']
		if self.data.type.code == gdb.TYPE_CODE_STRUCT:
			self.data = self.data['array']
			self.data = self.data.cast(self.innerType.pointer())
			
	class _iterator(_MatrixEntryIterator):
		def __init__ (self, rows, cols, dataPtr, rowMajor):
			super(EigenMatrixPrinter._iterator, self).__init__(rows, cols, rowMajor)

			self.dataPtr = dataPtr

		def __next__(self):
			
			row, col = super(EigenMatrixPrinter._iterator, self).__next__()
			
			item = self.dataPtr.dereference()
			self.dataPtr = self.dataPtr + 1
			if (self.cols == 1): #if it's a column vector
				return ('[%d]' % (row,), item)
			elif (self.rows == 1): #if it's a row vector
				return ('[%d]' % (col,), item)
			return ('[%d,%d]' % (row, col), item)
			
	def children(self):
		
		return self._iterator(self.rows, self.cols, self.data, self.rowMajor)
		
	def to_string(self):
		return "Eigen::%s<%s,%d,%d,%s> (data ptr: %s)" % (self.variety, self.innerType, self.rows, self.cols, "RowMajor" if self.rowMajor else  "ColMajor", self.data)

class EigenSparseMatrixPrinter:
	"Print an Eigen SparseMatrix"

	def __init__(self, val):
		"Extract all the necessary information"

		type = val.type
		if type.code == gdb.TYPE_CODE_REF:
			type = type.target()
		self.type = type.unqualified().strip_typedefs()
		tag = self.type.tag
		regex = re.compile('\<.*\>')
		m = regex.findall(tag)[0][1:-1]
		template_params = m.split(',')
		template_params = [x.replace(" ", "") for x in template_params]

		self.options = 0
		if len(template_params) > 1:
			self.options = template_params[1];
		
		self.rowMajor = (int(self.options) & 0x1)
		
		self.innerType = self.type.template_argument(0)
		
		self.val = val

		self.data = self.val['m_data']
		self.data = self.data.cast(self.innerType.pointer())

	class _iterator(_MatrixEntryIterator):
		def __init__ (self, rows, cols, val, rowMajor):
			super(EigenSparseMatrixPrinter._iterator, self).__init__(rows, cols, rowMajor)

			self.val = val
			
		def __next__(self):
			
			row, col = super(EigenSparseMatrixPrinter._iterator, self).__next__()
				
			# repeat calculations from SparseMatrix.h:
			outer = row if self.rowMajor else col
			inner = col if self.rowMajor else row
			start = self.val['m_outerIndex'][outer]
			end = ((start + self.val['m_innerNonZeros'][outer]) if self.val['m_innerNonZeros'] else
			       self.val['m_outerIndex'][outer+1])

			# and from CompressedStorage.h:
			data = self.val['m_data']
			if start >= end:
				item = 0
			elif (end > start) and (inner == data['m_indices'][end-1]):
				item = data['m_values'][end-1]
			else:
				# create Python index list from the target range within m_indices
				indices = [data['m_indices'][x] for x in range(int(start), int(end)-1)]
				# find the index with binary search
				idx = int(start) + bisect_left(indices, inner)
				if ((idx < end) and (data['m_indices'][idx] == inner)):
					item = data['m_values'][idx]
				else:
					item = 0

			return ('[%d,%d]' % (row, col), item)

	def children(self):
		if self.data:
			return self._iterator(self.rows(), self.cols(), self.val, self.rowMajor)

		return iter([])   # empty matrix, for now


	def rows(self):
		return self.val['m_outerSize'] if self.rowMajor else self.val['m_innerSize']

	def cols(self):
		return self.val['m_innerSize'] if self.rowMajor else self.val['m_outerSize']

	def to_string(self):

		if self.data:
			status = ("not compressed" if self.val['m_innerNonZeros'] else "compressed")
		else:
			status = "empty"
		dimensions  = "%d x %d" % (self.rows(), self.cols())
		layout      = "row" if self.rowMajor else "column"

		return "Eigen::SparseMatrix<%s>, %s, %s major, %s" % (
			self.innerType, dimensions, layout, status )

class EigenQuaternionPrinter:
	"Print an Eigen Quaternion"
	
	def __init__(self, val):
		"Extract all the necessary information"
		# The gdb extension does not support value template arguments - need to extract them by hand
		type = val.type
		if type.code == gdb.TYPE_CODE_REF:
			type = type.target()
		self.type = type.unqualified().strip_typedefs()
		self.innerType = self.type.template_argument(0)
		self.val = val
		
		# Quaternions have a struct as their storage, so we need to walk through this
		self.data = self.val['m_coeffs']['m_storage']['m_data']['array']
		self.data = self.data.cast(self.innerType.pointer())
			
	class _iterator:
		def __init__ (self, dataPtr):
			self.dataPtr = dataPtr
			self.currentElement = 0
			self.elementNames = ['x', 'y', 'z', 'w']
			
		def __iter__ (self):
			return self
	
		def next(self):
			return self.__next__()  # Python 2.x compatibility

		def __next__(self):
			element = self.currentElement
			
			if self.currentElement >= 4: #there are 4 elements in a quanternion
				raise StopIteration
			
			self.currentElement = self.currentElement + 1
			
			item = self.dataPtr.dereference()
			self.dataPtr = self.dataPtr + 1
			return ('[%s]' % (self.elementNames[element],), item)
			
	def children(self):
		
		return self._iterator(self.data)
	
	def to_string(self):
		return "Eigen::Quaternion<%s> (data ptr: %s)" % (self.innerType, self.data)

def build_eigen_dictionary ():
	pretty_printers_dict[re.compile('^Eigen::Quaternion<.*>$')] = lambda val: EigenQuaternionPrinter(val)
	pretty_printers_dict[re.compile('^Eigen::Matrix<.*>$')] = lambda val: EigenMatrixPrinter("Matrix", val)
	pretty_printers_dict[re.compile('^Eigen::SparseMatrix<.*>$')] = lambda val: EigenSparseMatrixPrinter(val)
	pretty_printers_dict[re.compile('^Eigen::Array<.*>$')]  = lambda val: EigenMatrixPrinter("Array",  val)

def register_eigen_printers(obj):
	"Register eigen pretty-printers with objfile Obj"

	if obj == None:
		obj = gdb
	obj.pretty_printers.append(lookup_function)

def lookup_function(val):
	"Look-up and return a pretty-printer that can print va."
	
	type = val.type
	
	if type.code == gdb.TYPE_CODE_REF:
		type = type.target()
	
	type = type.unqualified().strip_typedefs()
	
	typename = type.tag
	if typename == None:
		return None
	
	for function in pretty_printers_dict:
		if function.search(typename):
			return pretty_printers_dict[function](val)
	
	return None

pretty_printers_dict = {}

build_eigen_dictionary ()

```
**Reference**
- https://gitlab.com/libeigen/eigen/-/blob/master/debug/gdb/printers.py
- https://codeantenna.com/a/RG3wc1ptFn