import json
import logging

from PyQt4.QtCore import SIGNAL
from PyQt4 import QtGui
import anki, aqt, re
import os


class BczHandler():

    def __init__(self):

        self.base_path = 'your_path/baicizhan_zpk-master/extra_zpk/'
        self.json_content_file = 'your_path/baicizhan_zpk-master/mata-json-content.txt'
        self.file = open(self.json_content_file, 'r')
        self.json_content = self.file.readlines()
        self.file_list_file_path = 'your_path/baicizhan_zpk-master/extra_zpk_result.txt'
        self.file_list = open(self.file_list_file_path, 'r')
        self.file_list_content = self.file_list.readlines()
        self.log_file_path = 'your_path/logs/anki/bcz_handler.log'
        self.log = self._setup_logger(self.log_file_path)
        self.shortCut = 'Ctrl + C'

        def setup_Menu(_browser):
            self.anki_media_path = os.path.join(_browser.mw.pm.profileFolder(), "collection.media/")
            menu = QtGui.QMenu('BczHandler', _browser.form.menubar)
            _browser.form.menubar.addMenu(menu)

            def append_Munu(_text, how):
                action = QtGui.QAction(_text, menu)
                # set shot cut
                from PyQt4.QtGui import QKeySequence
                action.setShortcut(QKeySequence(self.shortCut))
                _browser.connect(action, SIGNAL('triggered()'),
                                 lambda s=_browser: self._run(s,))
                menu.addAction(action)

            append_Munu('Import selected word(s)...', 'useless parameter')

        anki.hooks.addHook('browser.setupMenus', setup_Menu, )

    def _run(self, _browser):
        _notes = [_browser.mw.col.getNote(note_id) for note_id in _browser.selectedNotes()]
        if not _notes:
            aqt.utils.showWarning('Please select one/some item(s).', _browser)
            return
        _browser.model.beginReset()
        for note in _notes:
            word = None
            try:
                word = unicode.strip(note['Front'])
                self.log.info('Processing the word [%s]', word)

                if 'baicizhan-mean_cn' not in note.keys():
                    self.log.info('The word [%s] does not have field of "baicizhan-mean_cn"', word)
                    continue

                if 'baicizhan-mean_en' not in note.keys():
                    self.log.info('The word [%s] does not have field of "baicizhan-mean_en"', word)
                    continue

                if 'reminder-baicizhan' not in note.keys():
                    self.log.info('The word [%s] does not have field of "reminder-baicizhan"', word)
                    continue

                data = self._get_meta_by_word(word)
                if data is None:
                    self.log.info('The word [%s] does not exist in bai-ci-zhan', word)
                    continue


                if note['baicizhan-mean_cn'] is not None and note['baicizhan-mean_cn'] != '':
                    self.log.info('The word [%s] had been added Chinese translation', word)
                else:
                    if 'mean_cn' in data.keys():
                        note['baicizhan-mean_cn'] = data['mean_cn']
                    else:
                        self.log.info('The word [%s] does not have Chinese translation', word)

                if note['baicizhan-mean_en'] is not None and note['baicizhan-mean_en'] != '':
                    self.log.info('The word [%s] had been added English translation', word)
                else:
                    if 'mean_en' in data.keys():
                        note['baicizhan-mean_en'] = data['mean_en']
                    else:
                        self.log.info('The word [%s] does not have English translation', word)

                if note['reminder-baicizhan'] is not None and note['reminder-baicizhan'] != '':
                    self.log.info('The word [%s] had been added image', word)
                else:
                    if 'image_file' in data.keys():
                        self._copy_files(data)
                        content = self._get_anki_content(data)
                        self.log.info('Content: %s' % content)
                        note['reminder-baicizhan'] = content
                    else:
                        self.log.info('The word [%s] does not have image', word)

                if note['word_etyma'] is not None and note['word_etyma'] != '':
                    self.log.info('The word [%s] had been added etyma', word)
                else:
                    if 'word_etyma' in data.keys():
                        note['word_etyma'] = data['word_etyma']
                    else:
                        self.log.info('The word [%s] does not have etyma', word)

                note.flush()
            except:
                self.log.info('Exception occurs, when handle note of the word [%s]', word)
                import traceback
                self.log.info(traceback.format_exc())

        _browser.model.endReset()

    def _get_meta_by_word(self, word):
        for each_line in self.json_content:
            data = json.loads(each_line)
            if word == data['word']:
                return data

    def _process_base_path(self, data):
        return self.base_path + 'zp_' + str(data['topic_id']) + '_'\
               + str(data['word_level_id']) + '_'\
               + str(data['tag_id']) + '_' + '*_*/'

    def _get_img_name(self, data):
        return data['image_file']

    def _get_img_path(self, data):
        file_name = self._get_img_name(data)
        return self._process_base_path(data) + file_name

    def _get_sentence(self, data):
        return data['sentence']

    def _get_sentence_trans(self, data):
        return data['sentence_trans']

    def _get_sentence_audio_name(self, data):
        return data['sentence_audio']

    def _get_sentence_audio_path(self, data):
        return self._process_base_path(data) + self._get_sentence_audio_name(data)

    # https://www.techbeamers.com/python-copy-file/
    def _copy_files(self, data):
        img_dest = self.anki_media_path + self._get_img_name(data)
        import os
        shell_cp_img = 'cp %s "%s"' % (self._get_img_path(data), img_dest)
        self.log.info('Shell: %s' % shell_cp_img)
        self.log.info(os.system('ls %s' % self._get_img_path(data)))
        self.log.info(os.system(shell_cp_img))

        sentence_audio_dest = self.anki_media_path + self._get_sentence_audio_name(data)
        shell_cp_sentence_audio = 'cp %s "%s"' % (self._get_sentence_audio_path(data), sentence_audio_dest)
        self.log.info('Shell: %s' % shell_cp_sentence_audio)
        self.log.info(os.system('ls %s' % self._get_sentence_audio_path(data)))
        self.log.info(os.system(shell_cp_sentence_audio))

    def _setup_logger(self, filename='bcz_handler.log'):
        log = logging.getLogger(__name__)

        logging.basicConfig(filename=filename,
                            level=logging.INFO,
                            format='%(asctime)s - %(name)s - %(threadName)s -  %(levelname)s - %(message)s')

        return log

    def _get_anki_content(self, data):

        return '<img src="%s" /><div>%s[sound:%s]</div><div>%s</div>' \
                  % (self._get_img_name(data),
                     self._get_sentence(data),
                     self._get_sentence_audio_name(data),
                     self._get_sentence_trans(data))


BczHandler()
